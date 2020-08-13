- Start Date: 2020-08-13
- Project: [React Native with MOXY](https://github.com/moxystudio/react-native-with-moxy)
- Implementation PR: (leave this empty)

# Summary

The feature is an abstraction layer to work with different mobile error tracking solutions. The idea here is to have a common API that will be mapped to the particular API of the chosen error tracking solution.

# Basic example

Capturing an error:

```js
import { captureError } from "../shared/utils/error-tracking";

try {
  aFunctionThatMightFail();
} catch (err) {
  captureError(err);
}
```

Throwing a native crash:

```js
import { nativeCrash } from "../shared/utils/error-tracking";

nativeCrash();
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

However we might decide to use another error tracking solution later, e.g. [Firebase Crashlytics](https://firebase.google.com/products/crashlytics). By doing so we would need to go through all our codebase and change all Sentry's methods to their Crashlytics' counterparts, which would be an unnecessary work.

```js
try {
  aFunctionThatMightFail();
} catch (err) {
  crashlytics().recordError(err);
}
```

What we would like to to instead is mapping our own API to the one from our current error tracking solution. Basically we would only need to change a few lines in a single file.
