- Start Date: 2020/01/11
- Project: [`@moxy/eslint-config`](https://github.com/moxystudio/eslint-config)
- Implementation PR: (leave this empty)

# Summary

Create several eslint config packages instead of having a single package with multiple addons.

# Motivation

Currently, [`@moxy/eslint-config`](https://github.com/moxystudio/eslint-config) has the concept of addons that you use to compose your eslint configuration. As an example, there's a `react` addon that enables [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react) and customizes its rules.

However, as the time passed by, `@moxy/eslint-config` has become a monolith with a fairly large set of addons. The trend is for the number of addons to continue to increase while we adopt new technologies, frameworks and libraries.

The monolith has several disadvantages:

- The dependencies for all addons are installed, even if we just use or two addons.
- It's becoming difficult to understand which addons we need to activate for a particular project.
- It's becoming hard to maintain everything, especially tests.

⚠️ To make things worst, the upcoming [`babel-eslint`](https://github.com/babel/babel-eslint) major version, which we use for the `babel-parser` addon, defines `@babel/core` as a peer dependency. Projects that don't use Babel will have to ignore the npm warning: `requires a peer of babel-eslint@^xxx but none was installed`.

# Detailed design

My proposal is to split `@moxy/eslint-config` into several packages, under the same repository (mono-repo).

## Base configs

Base configs sets the foundations in terms of eslint settings. After analyzing, there's currently three types of base configs amongst our packages:

- Node.js only projects
- Browser only projects
- Isomorphic projects that work both in Node.js and Browser environments

### `@moxy/eslint-config-node`

**Base** config for Node.js based projects (e.g.: APIs) and libraries.

- Configures all our base rules
- Sets ECMAScript version `es9` by default
- Turns on `node` environment

```js
{
    "extends": [
        "@moxy/eslint-config-node",
    ]
}
```

Other ECMAScript versions might be requested by adding `/<version>.js`, like so:

```js
{
    "extends": [
        "@moxy/eslint-config-node/es8",
    ]
}
```

ℹ️ Future versions of Node.js will have support for ES Modules, see https://nodejs.org/api/esm.html. Once that becomes available, we may upgrade `@moxy/eslint-config-node` to use ES Modules. If necessary, we may create `@moxy/eslint-config-node-legacy` for older projects, while keeping the base rules up to date.

### `@moxy/eslint-config-browser`

**Base** config for browser based projects and libraries.

- Configures all our base rules
- Sets ECMAScript version to latest
- Turns on `browser` environment
- Turns on `es-modules`
- Turns on `babel-parser`

```js
{
    "extends": [
        "@moxy/eslint-config-browser"
    ]
}
```

As listed above, these type of projects will be using Babel. Since Babel will be taking care of making code compatibility with target browsers, we can always use the latest ECMAScript version.

If the project uses `workers` or `service-workers`, you will have to enable it manually via the `env` key.

### `@moxy/eslint-config-isomorphic`

**Base** config for isomorphic projects (e.g.: Next.js) and libraries.

- Configures all our base rules
- Sets ECMAScript version `es9`
- Turns on `node` and `browser` environments
- Turns on `es-modules`
- Turns on `babel-parser`

```js
{
    "extends": [
        "@moxy/eslint-config-isomorphic"
    ]
}
```

As listed above, these type of projects will be using Babel. Since Babel will be taking care of making code compatibility with target browsers, we can always use the latest ECMAScript version.

If the project uses `workers` or `service-workers`, you will have to enable it manually via the `env` key.

## Enhancers for the base config

### `@moxy/eslint-config-jest`

- Enables `eslint-plugin-jest`
- Configures rules specific to Jest

```js
{
    "extends": [
        "@moxy/eslint-config-node",
        "@moxy/eslint-config-jest"
    ]
}
```

### `@moxy/eslint-config-react`

- Enables `eslint-plugin-react`
- Configures rules specific to react

```js
{
    "extends": [
        "@moxy/eslint-config-isomorphic",
        "@moxy/eslint-config-react"
    ]
}
```

### `@moxy/eslint-config-vue`

- Enables `eslint-plugin-vue`
- Configures rules specific to vue

```js
{
    "extends": [
        "@moxy/eslint-config-browser",
        "@moxy/eslint-config-vue"
    ]
}
```

# Drawbacks

- Users have to install more than one eslint config related package
- We need to become familiar with Lerna

ℹ️ Getting familiar with Lerna might actually be a good thing as it will be useful for future projects.

# Alternatives

1. We can keep using the same strategy: a monolithic package.
2. We can still follow this strategy, with separate repositories and without Lerna.

# Adoption strategy

The adoption can be made gradually, by updating projects once we need to contribute to them.

We may also leverage `screpto` to automate this for us.

# Unresolved questions

1. Should the base configs started with `@moxy/eslint-config-base-`, e.g.: `@moxy/eslint-config-base-node`?
