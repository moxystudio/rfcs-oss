- Start Date: 2020-08-13
- Project: [React Native with MOXY](https://github.com/moxystudio/react-native-with-moxy)
- Implementation PR:

# Summary

The feature is an abstraction layer to work with different mobile error tracking solutions. The idea here is to have a common API that will be mapped to the particular API of the chosen error tracking solution.

# Basic example

Capturing an error:

```js
import createErrorTracking from "@moxy/react-native-sentry";

const errorTracking = createErrorTracking({ enabled: true });

try {
  aFunctionThatMightFail();
} catch (err) {
  errorTracking.captureError(err);
}
```

# Motivation

Let's suppose we were using [Sentry](https://sentry.io) as our solution. So, we would be able to capture an error using Sentry's `captureException` method.

```js
try {
  aFunctionThatMightFail();
} catch (err) {
  Sentry.captureException(err);
}
```

However we might decide to use another error tracking solution later, e.g. [Firebase Crashlytics](https://firebase.google.com/products/crashlytics). By doing so we would need to go through all our codebase and change all Sentry's methods to their Crashlytics' counterparts, which would becumbersome and time consuming work.

```js
try {
  aFunctionThatMightFail();
} catch (err) {
  crashlytics().recordError(err);
}
```

What we would like to to instead is mapping our own API to the one from our current error tracking solution. Basically we would only need to import another module.

Another important motivation is to add the ability to toggle the error reporting whenever we want to. Unfortunately, some solutions don't have this feature built-in, so we will create a way to handle this inside the abstraction layer.

# Detailed design

Example for Sentry:

```js
import * as Sentry from "@sentry/react-native";

const captureError = Sentry.captureException;

// level should be: fatal, error, warning, info, or debug.
const addBreadcrumb = ({ scope, message, level = "info" }) =>
  Sentry.addBreadcrumb({
    category: scope,
    message,
    level,
  });

const setUserId = (level) =>
  Sentry.configureScope((scope) => scope.setLevel(level));

// You can set the severity of an event to one of five values: fatal, error, warning, info, and debug. error is the default, fatal is the most severe, and debug is the least severe.

const setLevel = (id) =>
  Sentry.configureScope((scope) => scope.setUser({ id }));

const createErrorTracking = ({ enabled = false } = {}) => {
  let isTrackingEnabled = enabled;

  const createTrackingFn = (fn) => (...args) => {
    if (!isTrackingEnabled) {
      return Promise.resolve();
    }

    return fn(...args);
  };

  const enable = (enabled) => {
    isTrackingEnabled = enabled;
  };

  return {
    captureError: createTrackingFn(captureError),
    addBreadcrumb: createTrackingFn(addBreadcrumb),
    setUserId: createTrackingFn(setUserId),
    setLevel: createTrackingFn(setLevel),
    enable,
  };
};

export default createErrorTracking;
```

Example for Firebase Crashlytics:

```js
import crashlytics from "@react-native-firebase/crashlytics";

const captureError = crashlytics().recordError;

const setUserId = crashlytics().setUserId;

const setLevel = (level) => crashlytics().setAttributes({ level });

// level should be: fatal, error, warning, info, or debug.
const addBreadcrumb = ({ scope, message, level = "info" }) =>
  crashlytics().setAttributes({ category: scope, message, level });

const createErrorTracking = ({ enabled = false } = {}) => {
  let isTrackingEnabled = enabled;

  const createTrackingFn = (fn) => (...args) => {
    if (!isTrackingEnabled) {
      return Promise.resolve();
    }

    return fn(...args);
  };

  const enable = (enabled) => {
    isTrackingEnabled = enabled;
  };

  return {
    captureError: createTrackingFn(captureError),
    addBreadcrumb: createTrackingFn(addBreadcrumb),
    setLevel: createTrackingFn(setLevel),
    setUserId: createTrackingFn(setUserId),
    enable,
  };
};

export default createErrorTracking;
```

## Usage examples

```js
import createErrorTracking from "@moxy/react-native-sentry";

const errorTracking = createErrorTracking({ enabled: true });

try {
  aFunctionThatMightFail();
} catch (err) {
  errorTracking.captureError(err);
}
```

## Regarding the initialization

In my opinion the initialization should not be abstracted, since Sentry and Firebase Crashlytics have quite differente ways of being initialized.

- Sentry:

```js
import * as Sentry from '@sentry/react-native';

Sentry.init({
    dsn: // Sentry's DSN,
});
```

- Firebase Crashlytics:

```js
import crashlytics from "@react-native-firebase/crashlytics";

// you can already use it inside the file
```

# Drawbacks

The only drawback I can see is the initial effort to design the solution, since the maintenance should be really simple.

# Alternatives

The alternative would be using the API provided directly from the error tracking solutions. However, this is not a good idea as I've already commented about in the "Motivation" section.

# Adoption strategy

To integrate those modules `@moxy/react-native-{solution}`, where `solution` would be the name of the chosen error tracking solution (it could be `sentry` for instance), one should start by installing it:

```sh
npm i @moxy/react-native-sentry

# yarn add @moxy/react-native-sentry
```

On the `src/app/App.js` file, your should initialize Sentry:

```js
import * as Sentry from '@sentry/react-native';

Sentry.init({
  dsn: // Sentry's DSN,
});
```

Then you will be able to use the `@moxy/react-native-sentry` module through your entire `App` tree:

Suppose there is a `Button.js` component:

```js
import React, { useCallback } from "react";
import { TouchableOpacity, Text } from "react-native";
import createErrorTracking from "@moxy/react-native-sentry";

const errorTracking = createErrorTracking({ enabled: true });

const aFunctionThatMightFail = () => {
  throw new Error("Failed");
};

const Button = () => {
  const onPress = useCallback(() => {
    try {
      aFunctionThatMightFail();
    } catch (err) {
      errorTracking.captureError(err);
    }
  }, []);

  return (
    <TouchableOpacity onPress={onPress}>
      <Text>Press Here</Text>
    </TouchableOpacity>
  );
};
```

Note that it's possible to enable or disable the error tracking as you wish, so it would be possible to:

```js
import React, { useState, useCallback } from "react";
import { View, ouchableOpacity, Text } from "react-native";
import createErrorTracking from "@moxy/react-native-sentry";

const ToggleErrorTracking = () => {
  const [enabled, setEnabled] = useState(true);

  const onPress = useCallback(() => {
    setEnabled((state) => !state);
  }, [setEnabled]);

  const errorTracking = createErrorTracking({ enabled });

  return (
    <View>
      <TouchableOpacity onPress={onPress}>
        <Text>Toggle Error Tracking</Text>
      </TouchableOpacity>

      <Text>Error tracking is {enabled ? "enabled" : "disabled"}</Text>
    </View>
  );
};
```
