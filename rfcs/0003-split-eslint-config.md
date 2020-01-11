- Start Date: 2020/01/11
- Project: [`@moxy/eslint-config`](https://github.com/moxystudio/eslint-config)
- Implementation PR: (leave this empty)

# Summary

Create several eslint config packages instead of having a single package with multiple addons.

# Motivation

Currently, [`@moxy/eslint-config`](https://github.com/moxystudio/eslint-config) has the concept of addons that you use to compose your eslint configuration.
As an example, there's a `react` addon that enables [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react) and customizes its rules.

However, as the time passed by, `@moxy/eslint-config` has become a monolith with a fairly large set of addons. The trend is for the number of addons to continue to increase while we adopt new technologies, frameworks and libraries.

The monolith has several disadvantages:

- The dependencies for all addons are installed, even if we just use or two addons.
- It's becoming difficult to understand which addons we need to activate for a particular project.
- It's becoming hard to maintain everything, especially tests.

# Detailed design

My proposal is to split `@moxy/eslint-config` into several packages, under the same repository (mono-repo).

### `@moxy/eslint-config-base`

- Defines the base JS version (es8, es9)
- Turns on `browser` and/or `node`
- Turns on `babel-parser`
- Turns on `es-modules`

This can be done like so:

```js
{
    "extends": [
        "@moxy/eslint-config-base/es9",
        "@moxy/eslint-config-base/node",
        "@moxy/eslint-config-base/browser",
        "@moxy/eslint-config-base/es-modules",
        "@moxy/eslint-config-base/babel-parser"
    ]
}
```

To make this simpler, we may define **presets for common cases**. At the moment, there are two common cases: **node.js** & **web** based projects:

```js
{
    "extends": [
        // equivalent to es9 & node
        "@moxy/eslint-config-base/preset-node"
    ]
}
```

```js
{
    "extends": [
        // equivalent to es9, browser, node, es-modules & babel-parser
        "@moxy/eslint-config-base/preset-web"  
    ]
}
```

### `@moxy/eslint-config-jest`

- Enables `eslint-plugin-jest`
- Configures rules specific to Jest

```js
{
    "extends": [
        "@moxy/eslint-config-base/preset-web",
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
        "@moxy/eslint-config-base/preset-web",
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
        "@moxy/eslint-config-base/preset-web",
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

N/A.
