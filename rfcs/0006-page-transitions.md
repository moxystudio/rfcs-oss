- Start Date: 2020-01-13
- Project: [next-with-moxy](https://github.com/moxystudio/next-with-moxy)
- Reference Issues: N/A
- Implementation PR: N/A

# Summary

This proposal is to add full page transitions to the `next-with-moxy` boilerplate, either as a guide and easy-to-use feature (recipe) or an out-of-the-box feature (by including it in the source code).

# Basic example

We will use [Motion](https://www.framer.com/motion/) by Framer, an open-source and production-ready motion library for React on the web.

## Installation

First, install the package:

```sh
$ npm install --save framer-motion
```

## Component hierarchy

To animate components as they're removed from the React tree, we need an [AnimatePresence](https://www.framer.com/api/motion/animate-presence/) component that wraps motion components (`Transition` in this case).

In our [`App.js`](https://github.com/moxystudio/next-with-moxy/blob/master/www/app/App.js) file:

```js
render() {
    const { Component, pageProps, router } = this.props;

    return (
        <>
            <Head>
                {/* ... */}
            </Head>
            <KeyboardOnlyOutlines>
                <AnimatePresence exitBeforeEnter>
                    <Transition key={ router.route }>
                        <Component { ...pageProps } />
                    </Transition>
                </AnimatePresence>
            </KeyboardOnlyOutlines>
        </>
    );
}
```

## Motion components

`Transition` is a [motion component](https://www.framer.com/api/motion/component/) that contains the animation to display when switching pages. A motion component works like their HTML counterparts, but offer some props that allow you to:

- Declaratively or imperatively animate components.
- Add drag, pan, hover and tap gestures.
- Respond to gestures with animations.
- Deeply animate throughout React trees via variants.

A good place to put these would be in the [`www/shared/components`](https://github.com/moxystudio/next-with-moxy/tree/master/www/shared/components), in a *transitions* folder for example.

```js
import React from 'react';
import PropTypes from 'prop-types';
import { motion } from 'framer-motion';

const variants = {
    initial: {
        opacity: 0,
    },
    enter: {
        opacity: 1,
        x: [600, 0],
        transition: {
            type: 'tween',
            duration: 0.8,
        },
    },
    exit: {
        opacity: 0,
        y: 300,
        transition: {
            type: 'tween',
            duration: 0.5,
        },
    },
};

const Transition = ({ children }) => (
    <motion.div
        variants={ variants }
        initial="initial"
        animate="enter"
        exit="exit">
        { children }
    </motion.div>
);

Transition.propTypes = {
    children: PropTypes.node.isRequired,
};

export default Transition;
```

The `exit` prop is what defines the animation to use when the component is unmounted, as opposed to the `animate` prop which defines the values to animate to. Learn more about the accepted props in the [documentation](https://www.framer.com/api/motion/component/#props).

The [`variants`](https://www.framer.com/api/motion/animation/#variants) define animation states and organizes them by name. They allow you to control animations throughout a component tree by switching a single `animate` prop.

# Motivation

Page transitions are animated transitions between pages that can improve the user's context by maintaining their attention and providing visual continuity.

It also reduces cognitive load, as Sarah Drasner [pointed out](https://css-tricks.com/native-like-animations-for-page-transitions-on-the-web/):

 > Transitioning between two states can reduce cognitive load for your user, as when someone is scanning a page, they have to create a mental map of everything that's contained on it. When we move from page to page, the user has to remap that entire space. If an element is repeated on several pages but altered slightly, it mimics the experience we've been biologically trained to expect — no one just pops into a room or changes suddenly; they transition from another room into this one.

When done well they can be aesthetically pleasing, reinforce branding and even change the entire feel of a website.

# Detailed design

Just follow the steps covered in the [basic example](#basic-example) section.

# Drawbacks

- Page transitions animations must be coded in JavaScript.
- Adding more dependencies to the project.

# Alternatives

Check the [first version of this RFC](https://github.com/moxystudio/rfcs-oss/pull/4) that had `Highway` and `next-page-transitions` as contenders, but as both had some major drawbacks they got dropped.

# Adoption strategy

There are two approaches we can take with the page transitions:

1. Create a recipe. As the steps are easy to follow, a recipe would be recommended for this.
2. Include Framer Motion in the source code. As full page transitions probably won't be used in every project, I think we should avoid this approach.

# Unresolved questions

In the [Basic example](#basic-example) section, every page component is wrapped by the `Transition` component, which means that every route will have the same *in* and *out* animation.

To have different animations per route, we can either wrap each page with its custom `Transition` component or check the current route and change the component to use on the fly.

I'd say that we should not standardize this and give the developer the freedom to do as he pleases.
