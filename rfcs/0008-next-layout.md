- Start Date: 2020-01-28
- Project: `@moxy/next-layout`
- Reference Issues:
- Implementation PR: https://github.com/moxystudio/next-layout/pull/1

# Summary

Ability to define which layout to use per page, in Next.js projects.

# Basic example

Setup `<LayoutManager>` in your `pages/_app.js` component:

```js
import React from 'react';
import { LayoutManager } from '@moxy/next-layout';

const App = ({ Component, pageProps }) => (
    <LayoutManager
        Component={ Component }
        pageProps={ pageProps } />
);

export default App;
```

...and then use `withLayout` in your page components, e.g.: in `pages/about.js`:

```js
import React from 'react';
import { withLayout } from '@moxy/next-layout';
import { PrimaryLayout } from '../components';
import styles from './about.module.css';

const About = () => (
    <div className={ styles.about }>
        <h1>About</h1>
    </div>
);

export default withLayout(<PrimaryLayout variant="light" />)(About);
```

# Motivation

Next.js projects usually have the need to have one or more layouts. Layouts are the "shell" of your app and usually contain navigation elements, such as an header and a footer. In the ideal scenario, each page would be able to say which layout they want to use, including tweaking its properties dynamically, such as `variant="light"`. However, we also want to keep the layout persistent in the React tree, to avoid having to remount it every time a user navigate between pages.

Historically, projects have been overlooking the need of multiple layouts or the ability to change layout props between pages. They start off with a simple layout and only later they handle this need, often with poor and non-scalable solutions.

I propose to create a library called `@moxy/next-layout`, that solves the need for multi-layouts and changing layout props dynamically in a consistent and reusable way.

# Detailed design

`@moxy/next-layout` is a very simple library that exposes a `<LayoutManager>` component and a `withLayout` to be used in pages.

## &lt;LayoutManager&gt;

A component that selects the layout based on what pages specify. It keeps the layout persistent between page transitions whenever possible (e.g.: when the layout is the same).

Here's the list of props it supports:

### Component

Type: `ReactElementType`

The page component, which maps to your App `Component` prop.

### pageProps

Type: `object`

The page component props, which maps to your App `pageProps` prop.

### defaultLayout

Type: `ReactElement`

The default layout to be used when a child page doesn't explicitly set one.

```js
// pages/_app.js
import React from 'react';
import { LayoutManager } from '@moxy/next-layout';
import { PrimaryLayout } from '../components';

const App = ({ Component, pageProps }) => (
    <LayoutManager
        Component={ Component }
        pageProps={ pageProps }
        defaultLayout={ <PrimaryLayout /> } />
);

export default App;
```

### children

Type: `function`

A [render prop](https://reactjs.org/docs/render-props.html) to customize the rendering of the layout.

Its signature is `({ Layout, layoutProps, layoutKey, Component, pageProps, pageKey }) => <ReactElement>`, where:

- `Layout` is the layout React component that should be rendered
- `layoutProps` is the props that should be passed to the layout React component
- `layoutKey` is a unique string for the layout to be used as `key`
- `Component` is the page React component that should be rendered
- `pageProps` is the props that should be passed to the page React component, and already includes `setLayoutProps` if the page was wrapped with [`withLayout`](#withlayoutlayoutpage)
- `pageKey` is a unique string for the page to be used as `key`

Passing a custom `children` render prop is useful to add layout and page transitions. Here's an example that uses [`react-transition-group`](https://reactcommunity.org/react-transition-group/) to apply a simple fade transition between layouts and pages:

```js
// pages/_app.js
import React from 'react';
import { LayoutManager } from '@moxy/next-layout';
import { TransitionGroup, CSSTransition } from 'react-transition-group';
import { PrimaryLayout } from '../components';

const App = ({ Component, pageProps }) => (
    <LayoutManager
        Component={ Component }
        pageProps={ pageProps }
        defaultLayout={ <PrimaryLayout /> }>
        { ({ Layout, layoutProps, layoutKey, Component, pageProps, pageKey }) => (
            <TransitionGroup>
                <CSSTransition key={ layoutKey } classNames="fade">
                    <Layout { ...layoutProps }>
                        <TransitionGroup>
                            <CSSTransition key={ pageKey } classNames="fade">
                                <Component { ...pageProps } />
                            </CSSTransition>
                        </TransitionGroup>
                    </Layout>
                </CSSTransition>
            </TransitionGroup>
        ) }
    </LayoutManager>
);

export default App;
```

## withLayout(layout?)(Page)

Sets up a `Page` component with the ability to select which `layout` to use. Moreover, it injects a `setLayoutProps` prop so that you may dynamically update the layout props.

### layout

Type: `ReactElement` or `function`

The layout to use for the `Page`. Can either be a `ReactElement` or a function that returns it.

The function form is useful when page props affects layout props. It has the following signature: `(ownProps) => <ReactElement>`. Please note that the function only runs once to determine the layout and its initial props.

### Page

Type: `ReactElementType`

The page component to wrap.

### Injected setLayoutProps

Type: `function`

Allows to dynamically change the layout props. Has the following signature: `(updater | stateChange, callback?)`.

The behavior of `setLayoutProps` is exactly the same as `setState` of class components, supporting both an object or an updater function.

```js
// pages/about.js
import React, { useCallback } from 'react';
import { withLayout } from '@moxy/next-layout';
import { PrimaryLayout } from '../components';

import styles from './about.module.css';

const About = ({ setLayoutProps }) => {
    const handleSetToDark = useCallback(() => {
        setLayoutProps({ variant="dark" });
        // ..or setLayoutProps((layoutProps) => ({ variant="dark" }));
    }, [setLayoutProps]);

    return (
        <div className={ styles.about }>
            <h1>About</h1>
            <button onClick={ handleSetToDark }>Enable dark mode</button>
        </div>
    );
};

export default withLayout(<PrimaryLayout variant="light" />)(About);
```

# Drawbacks

N/A.

# Alternatives

I couldn't find any OSS library that was capable of doing what I'm proposing.

I also considered doing this directly in [`next-with-moxy`](https://github.com/moxystudio/next-with-moxy). However, it's valuable to have it in a separate package to make it easier to re-use, even outside Next.js projects.

# Adoption strategy

Add it to our boilerplate, so that new projects start using it. Whenever necessary, `@moxy/next-layout` can be added to older projects as well.

Furthermore, it seems to integrate well with [RFC 0006](https://github.com/moxystudio/rfcs-oss/pull/6) (page transitions).

# Unresolved questions

N/A.
