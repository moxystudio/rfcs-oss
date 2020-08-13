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

Another important motivation is the add the ability to toggle the error reporting whenever we want to. Unfortunately, some solutions don't have this feature built-in.

# Detailed design

I haven't started to work on the design yet.

# Drawbacks

The only drawback I can see is the initial effort to design the solution, since the maintenance should be really simple.

# Alternatives

The alternative would be using the API provided directly from the error tracking solutions. However, this is not a good idea as I've already commented about in the "Motivation" section.

# Adoption strategy

If we implement this proposal, the developers would need to check our own module which is probably going to be called `error-tracking.js` and see the available methods. I will probably add this to the RNWM documentation, to make its usage as clear as possible.

# Unresolved questions

- Detailed design
