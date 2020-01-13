- Start Date: 2020-01-13
- Project: [next-with-moxy](https://github.com/moxystudio/next-with-moxy)
- Reference Issues: N/A
- Implementation PR: N/A

# Summary

This proposal is to add full page transitions to the `next-with-moxy` boilerplate, either as a guide and easy-to-use feature (recipe) or an out-of-the-box feature (by including it in the source code).

# Basic example

We will use [Highway](https://highway.js.org/) by Dogstudio, a lightweight library to create the navigation animations.

## Installation

First, install the package:

```sh
$ npm install --save @dogstudio/highway
```

## Enabling Highway

In our [`App.js`](https://github.com/moxystudio/next-with-moxy/blob/master/www/app/App.js) file, we'll have to enable Highway:

```js
import Highway from '@dogstudio/highway/src/highway';
import { Slide, Fade } from '../shared/components/transitions';

class App extends NextApp {
    componentDidMount() {
        this.unregisterGoogleTracking = registerGoogleTracking(this.props.router);

        new Highway.Core({
            transitions: {
                '/contact': Fade,
                default: Slide,
            },
        });
    }
```

You can learn more about it [here](https://highway.js.org/get-started.html#core), but the gist of it is that the `transitions` object tells which [Transition component](https://highway.js.org/get-started.html#transitions) to use in which route. In this example, you can see that the `/contact` route will use the Fade component and every other route (default) will use the Slide one. We will get to it [in a moment](#transition-component).

## HTML Structure

Highway requires a [`data-router-wrapper`](https://highway.js.org/get-started.html#html-structure) to look inside for a [`data-router-view`](https://highway.js.org/get-started.html#html-structure) to update the content. So, still in our [`App.js`](https://github.com/moxystudio/next-with-moxy/blob/master/www/app/App.js) file:

```js
render() {
    const { Component, pageProps, router } = this.props;

    return (
        <>
            <Head>
                {/* ... */}
            </Head>
            <KeyboardOnlyOutlines>
                <div data-router-wrapper>
                    <div data-router-view={ router.route }>
                        <Component { ...pageProps } />
                    </div>
                </div>
            </KeyboardOnlyOutlines>
        </>
    );
}
```

## Transitions

The transitions are the managers of the animations to display or hide a page and each page can have its own custom transition. Read more about it [here](https://highway.js.org/get-started.html#transitions).

Every time a page is either displayed or hidden on navigation, the JS inside a transition will run. We have the `in` and `out` methods that should contain the animation to display and hide a `data-router-view`, respectively. These methods get an object as parameter with data:

- `to`: The data-router-view to display.
- `from`: The data-router-view to hide.
- `done`: The required callback method that has to be called once the animation is done.
- `trigger`: The triggered link, popstate or script.

A good place to put these would be in the [`www/shared/components`](https://github.com/moxystudio/next-with-moxy/tree/master/www/shared/components), in a *transitions* folder for example.

```js
import Highway from '@dogstudio/highway/src/highway';
import anime from 'animejs';

class Fade extends Highway.Transition {
    out({ from, done }) {
        anime({
            targets: from,
            opacity: [1, 0],
            duration: 1000,
            complete: () => done(),
        });
    }

    in({ from, to, done }) {
        from.remove();

        anime({
            targets: to,
            opacity: [0, 1],
            duration: 1000,
            complete: () => done(),
        });
    }
}

export default Fade;
```

# Motivation

Page transitions are animated transitions between pages that can improve the user's context by maintaining their attention and providing visual continuity.

It also reduces cognitive load, as Sarah Drasner [pointed out](https://css-tricks.com/native-like-animations-for-page-transitions-on-the-web/):

 > Transitioning between two states can reduce cognitive load for your user, as when someone is scanning a page, they have to create a mental map of everything that's contained on it. When we move from page to page, the user has to remap that entire space. If an element is repeated on several pages but altered slightly, it mimics the experience we've been biologically trained to expect — no one just pops into a room or changes suddenly; they transition from another room into this one.

When done well they can be aesthetically pleasing, reinforce branding and even change the entire feel of a website.

# Detailed design

Just follow the steps covered in the [basic example](#basic-example) section.

# Drawbacks

- Doesn't support server-side rendering (check the [unresolved questions](#unresolved-questions)).
- Doesn't support scroll restoration.
- Page transitions animations must be coded in JavaScript.
- Adding more dependencies to the project.

# Alternatives

Another library that has been considered is [next-page-transitions](https://github.com/illinois/next-page-transitions).

It has some points in favor, namely that it was built to work in Next.js and has built-in support for showing a loading indicator if the page component has to load data before it can be shown, although we can circumvent that with rendering that in the page itself for example. It also uses CSS for the animations.

Let's do a quick rundown of how it can be used.

In our [`App.js`](https://github.com/moxystudio/next-with-moxy/blob/master/www/app/App.js) file:

```js
import { PageTransition } from 'next-page-transitions';
import './App.css';

class App extends NextApp {
    // ...

    render() {
        const { Component, pageProps, router } = this.props;

        return (
            <>
                <Head>
                    {/* ... */}
                </Head>
                <KeyboardOnlyOutlines>
                    <PageTransition timeout={ 300 } classNames="page-transition">
                        <Component { ...pageProps } key={ router.route } />
                    </PageTransition>
                </KeyboardOnlyOutlines>
            </>
        );
    }
}
```

Note the `page-transition` className, when the key prop changes, the `PageTransition` component detects that and appends classes the same way [react-transition-group](https://github.com/reactjs/react-transition-group) does (it uses it as a dependency).

We add the CSS with the animations to match the classes:

```css
:global .page-transition-enter {
    opacity: 0;
}

:global .page-transition-enter-active {
    opacity: 1;
    transition: opacity 400ms;
}

:global .page-transition-exit {
    opacity: 1;
}

:global .page-transition-exit-active {
    opacity: 0;
    transition: opacity 400ms;
}
```

We have to use the `:global` specifier because the library [doesn't support CSS Modules](https://github.com/illinois/next-page-transitions/issues/7) and doesn't [spread the props when passing them to CSSTransition](http://reactcommunity.org/react-transition-group/css-transition#CSSTransition-prop-classNames).

Although this alternative may seem easier to use at first, it's not as flexible as Highway nor has the active development and roadmap of the latter.

# Adoption strategy

There are two approaches we can take with the page transitions:

1. Create a recipe. As the steps are easy to follow, a recipe would be recommended for this.
2. Include Highway in the source code. As full page transitions probably won't be used in every project, I think we should avoid this approach.

# Unresolved questions

Highway crashes on import because it doesn't support server-side rendering. We can either open a PR fixing that, fork the repo if it's not accepted or find a way to mock the window on the server-side.
