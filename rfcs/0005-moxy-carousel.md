- Start Date: 2020-01-21
- Project: [react-carousel](https://github.com/moxystudio/react-carousel)
- Reference Issues: [OFICINA-43](https://moxy.atlassian.net/browse/OFICINA-43)
- Implementation PR: <https://gitlab.com/moxystudio/inoveretail/inovretail-www/merge_requests/102>

# Summary

This proposal is to use [react-multi-carousel](https://github.com/YIZHUANG/react-multi-carousel) as a baseline to develop our own `Carousel` component. Comparing to other npm packages, namely [nuka-carousel](https://github.com/FormidableLabs/nuka-carousel), [react-slick](https://github.com/akiran/react-slick) and [material-auto-rotating-carousel](https://github.com/TeamWertarbyte/material-auto-rotating-carousel), it's the one that better fits our needs, which are:

- Infinite mode
- Center mode
- Control mode (arrows/dots)
- Custom animation
- AutoPlay mode
- Responsive
- Swipe to slide
- Mouse drag to slide
- Keyboard control to slide
- Custom styling
- Accessibility support
- Framework agnostic - *this is the only one not offered by `react-multi-carrousel`*

# Basic example

This example was made using [next-with-moxy](https://github.com/moxystudio/next-with-moxy/) boilerplate.

Preview:

![react-carousel](https://user-images.githubusercontent.com/8797405/72811982-9dc5dc80-3c58-11ea-9a26-aaed7a2e25c4.gif)

- `Home.js`

```jsx
import React from 'react';
import Carousel from '@moxy/react-carousel';
import classNames from 'classnames';

import styles from './Home.module.css';

const responsive = {
    desktop: {
        breakpoint: { max: 3000, min: 1024 },
        items: 3,
        slidesToSlide: 1, // Optional, default to 1.
        partialVisibilityGutter: 40,
    },
    tablet: {
        breakpoint: { max: 1024, min: 464 },
        items: 2,
        slidesToSlide: 2, // Optional, default to 1.
    },
    mobile: {
        breakpoint: { max: 464, min: 0 },
        items: 1,
        slidesToSlide: 1, // Optional, default to 1.
    },
};

const Slide = ({ className, title }) => (
    <div className={ classNames(styles.slide, className) }>{title}</div>
);

const slidesData = [
    { title: 'item 1', className: styles.firstSlide },
    { title: 'item 2', className: styles.secondSlide },
    { title: 'item 3', className: styles.thirdSlide },
    { title: 'item 4', className: styles.forthSlide },
];

const Home = () => (
    <Carousel
        swipeable={ false }
        draggable={ false }
        showDots
        responsive={ responsive }
        ssr // Means to render carousel on server-side.
        infinite
        autoplay={ false }
        autoPlaySpeed={ 1000 }
        keyBoardControl
        customTransition="all .5"
        transitionDuration={ 500 }
        containerClass={ styles.containerClass }
        removeArrowOnDeviceType={ ['tablet', 'mobile'] }
        dotListClass="custom-dot-list-style"
        centerMode
        itemClass={ styles.itemClass }
        activeItemClass={ styles.activeItemClass }>
        {slidesData.map((data) => (
            <Slide key={ data.title } { ...data } />
        ))}
    </Carousel>
);

export default Home;
```

- `Home.module.css`

```css
@import "../../shared/styles/variables";

.home {
    width: var(--home-width);
    height: var(--home-height);
    background-color: var(--color-white);
}

.containerClass {
    margin-top: 200px;

    & .itemClass {
        display: flex;
        justify-content: center;

        opacity: 0.3;

        & .slide {
            width: 100%;
            height: 200px;

            display: flex;
            justify-content: center;
            align-items: center;

            color: whitesmoke;
            font-weight: 700;
            text-transform: uppercase;
        }

        & .firstSlide {
            background-color: hotpink;
        }

        & .secondSlide {
            background-color: purple;
        }

        & .thirdSlide {
            background-color: darkcyan;
        }

        & .forthSlide {
            background-color: darksalmon;
        }
    }

    & .activeItemClass {
        opacity: 1;
    }
}
```

# Motivation

Everytime we need to implement a carousel/slider we are dependant on external libs, usually these libs meet the basic requirements, but eventually they will become problematic for more extended use cases.

It would be of great value to have our own implemented carousel/slider to prevent extended overhead time when implementing the wrapper for the external module.

# Detailed design

The original `react-multi-carousel` was lacking a way to style the active item(s) without overriding the package's CSS, so I added a new prop called `activeItemClass` that accepts a `class`.

More changes and/or additions will come after discussion about our needs.

# Drawbacks

- Depending on how many new features we will add, this will drastically diverge from upstream and thus it will require more effort to maintain;
- The original codebase is written in TypeScript (although I don't think this is technically a drawback, it should be noted);
- Not compatible with React Native;
- There are 13 contributors, but it seems that the author has been doing almost all of the work.

# Alternatives

- [nuka-carousel](https://github.com/FormidableLabs/nuka-carousel)
- [react-slick](https://github.com/akiran/react-slick)
- [material-auto-rotating-carousel](https://github.com/TeamWertarbyte/material-auto-rotating-carousel)

# Adoption strategy

If we implement this proposal we will need to have in mind that it will require some time to maintain it.

# Unresolved questions

I'm not sure about [removing the original `react-multi-carousel-item--active`](https://github.com/moxystudio/react-carousel/commit/4f7b87429775f18e455779de38d3b8d64647a688) as the default `class` for an active item. Maybe I should keep it and simply append the `class` given through the `activeItemClass`:

```jsx
<Carousel activeItemClass="xpto">
  // ...slides
</Carousel>
```

```html
<li class="react-multi-carousel-item react-multi-carousel-item--active xpto">...</li>
```
