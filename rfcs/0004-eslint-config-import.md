- Start Date: 2020-01-20
- Project: `@moxy/eslint-config-*`
- Reference Issues: [#79](https://github.com/moxystudio/eslint-config/pull/79)
- Implementation PR: (leave this empty)

# Summary

Add import rules to our code style so that we can automate some import order rules and standardize code practices, hence reducing code review issues.
Effectively, it would make the order and grouping of the imports deterministic.

# Motivation

Across projects, different approaches for ordering and organizing import statements in js files are used. This creates code inconsistency and non-standard code styling.

**Examples**


[Header.js](https://gitlab.com/moxystudio/thu/thu-japan-2020/blob/master/www/shared/components/header/Header.js)

```js
import React from 'react';
import PropTypes from 'prop-types';
import classNames from 'classnames';
// Components
import { LanguageSelector } from './components';
import SoundToggle from '../sound-toggle';
// Icons
import { LogoIcon } from '../icons';
// Styles
import styles from './Header.css';
```

[App.js](https://gitlab.com/moxystudio/thu/thu-japan-2020/blob/master/www/app/App.js)

```js
import React from 'react';
import NextApp from 'next/app';
import Head from 'next/head';
import keyboardOnlyOutlines from 'keyboard-only-outlines';
import registerGoogleTracking from './ga-tracking';
import { withNextIntlSetup } from '@moxy/next-intl';
import nextIntlConfig from '../../intl';

import favicon16 from '../shared/media/favicons/favicon-16x16.png';
import favicon32 from '../shared/media/favicons/favicon-32x32.png';
import favicon96 from '../shared/media/favicons/favicon-96x96.png';
import favicon192 from '../shared/media/favicons/favicon-192x192.png';

import SEO_DATA from './App.data.js';

import '../shared/styles/index.css';
```

Just like any other styling rule, ordering the import statements would benefit code-bases by making them consistent and reducing strain on code reviews and code style decisions.


# Detailed design

This RFC proposes an initial implementation of rules that will allow to improve the consistency in the import sections. These rules allows to:

> Group order in import statements.
> Order: Builtin, external, parent relative imports, local imports.

All of the applied rules would be autofixable, meaning we will not need to worry about ordering the imports manually to fix linting issues.

The implementation would rely on the existing eslint plugin: [eslint-plugin-import](https://github.com/benmosher/eslint-plugin-import).

### eslint-plugin-import rules

- [`import/order`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/order.md)
    - sort order will be the default group sort order: `builtin`, `external`, `parent`, `sibling`, `index`
        Example:
        ```js
        // 1. node "builtin" modules
        import fs from 'fs';
        import path from 'path';

        // 2. "external" modules
        import _ from 'lodash';
        import chalk from 'chalk';

        // 3. modules from a "parent" directory
        import foo from '../foo';
        import qux from '../../foo/qux';

        // 4. "sibling" modules from the same or a sibling's directory
        import bar from './bar';
        import baz from './bar/baz';

        // 5. "index" of the current directory
        import main from './';
        ```
    - force a new line between each import group with `'import/order': [1, { 'newlines-between': 'always' }]`
        Example:
        ```js
        import fs from 'fs';
        import path from 'path';

        import _ from 'lodash';
        import chalk from 'chalk';

        import qux from '../../foo/qux';
        import foo from '../foo';

        import bar from './bar';
        import baz from './bar/baz';

        import main from './';
        ```

- [`import/first`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/first.md): do not allow import statements to be defined outside the import section
- [`import/no-commonjs`](https://github.com/benmosher/eslint-plugin-import/blob/master/docs/rules/no-commonjs.md): replace `prefer-import` with `import/no-commonjs`, `import/no-amd` and `import/no-dynamic-require`.

The last two rules `import/first` and `import/no-commonjs` will replace the current module [`eslint-plugin-prefer-import`](https://www.npmjs.com/package/eslint-plugin-prefer-import).

## Examples

Bellow are some examples on how some existing code we have would look like if we adopted these new rules.

### Example 1: [Header.js](https://gitlab.com/moxystudio/thu/thu-japan-2020/blob/master/www/shared/components/header/Header.js)

**Original**

```js
import React from 'react';
import PropTypes from 'prop-types';
import classNames from 'classnames';
// Components
import { LanguageSelector } from './components';
import SoundToggle from '../sound-toggle';
// Icons
import { LogoIcon } from '../icons';
// Styles
import styles from './Header.css';
```

**With Sort Rules**

```js
import React from 'react';
import PropTypes from 'prop-types';
import classNames from 'classnames';

import { LogoIcon } from '../icons';
import SoundToggle from '../sound-toggle';

import { LanguageSelector } from './components';
import styles from './Header.css';
```

### Example 2: [App.js](https://gitlab.com/moxystudio/thu/thu-japan-2020/blob/master/www/app/App.js)

**Original**

```js
import React from 'react';
import NextApp from 'next/app';
import Head from 'next/head';
import keyboardOnlyOutlines from 'keyboard-only-outlines';
import registerGoogleTracking from './ga-tracking';
import { withNextIntlSetup } from '@moxy/next-intl';
import nextIntlConfig from '../../intl';

import favicon16 from '../shared/media/favicons/favicon-16x16.png';
import favicon32 from '../shared/media/favicons/favicon-32x32.png';
import favicon96 from '../shared/media/favicons/favicon-96x96.png';
import favicon192 from '../shared/media/favicons/favicon-192x192.png';

import SEO_DATA from './App.data.js';

import '../shared/styles/index.css';
```


**With Sort Rules**

```js
import React from 'react';
import NextApp from 'next/app';
import Head from 'next/head';
import keyboardOnlyOutlines from 'keyboard-only-outlines';
import { withNextIntlSetup } from '@moxy/next-intl';

import nextIntlConfig from '../../intl';
import favicon16 from '../shared/media/favicons/favicon-16x16.png';
import favicon32 from '../shared/media/favicons/favicon-32x32.png';
import favicon96 from '../shared/media/favicons/favicon-96x96.png';
import favicon192 from '../shared/media/favicons/favicon-192x192.png';
import '../shared/styles/index.css';

import registerGoogleTracking from './ga-tracking';
import SEO_DATA from './App.data.js';
```

### Example 3: [Home.js](https://gitlab.com/moxystudio/thu/thu-japan-2020/blob/master/www/pages/home/Home.js)

**Original**

```js
import React, { useState, useCallback, useRef } from 'react';
import PropTypes from 'prop-types';
import classnames from 'classnames';
import { useIntl } from 'react-intl';
// Components
import { Date, InteractiveForm, Intro, Loader, Place, StoryTellingEvent, Theme, Venue } from './components';
import {
    Thanks,
    Header,
    Link,
    ScrollingHint,
    Timeline,
    World,
    WaypointContentTransition,
    SoundToggle,
    JoinTribe,
} from '../../shared/components';
// Icons
import { LogoTextIcon, KagaCityIcon } from '../../shared/components/icons';
// Utils
import { getMessages } from '../../shared/utils';
import withViewPort from '../../shared/components/hocs/with-viewport';
// Data
import { externalLinks } from '../../shared/data';
// Styles
import styles from './Home.css';
```

**With Sort Rules**

```js
import React, { useState, useCallback, useRef } from 'react';
import PropTypes from 'prop-types';
import classnames from 'classnames';
import { useIntl } from 'react-intl';

import {
    Thanks,
    Header,
    Link,
    ScrollingHint,
    Timeline,
    World,
    WaypointContentTransition,
    SoundToggle,
    JoinTribe,
} from '../../shared/components';
import { LogoTextIcon, KagaCityIcon } from '../../shared/components/icons';
import { getMessages } from '../../shared/utils';
import withViewPort from '../../shared/components/hocs/with-viewport';
import { externalLinks } from '../../shared/data';

import { Date, InteractiveForm, Intro, Loader, Place, StoryTellingEvent, Theme, Venue } from './components';
import styles from './Home.css';
```

# Drawbacks

Adaptation to this new order and grouping of the imports might cause some confusion since styling is subjective.

# Alternatives

Enable more options to create deterministic sorting within each group: this would disable any custom sorting making the order of the imports always the same across projects.

# Adoption strategy

Incremental adoption would be the best approach since this change would be a breaking change.

# Unresolved questions

None
