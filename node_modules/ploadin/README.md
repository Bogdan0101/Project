# ploadin

[![Latest Version](https://img.shields.io/npm/v/ploadin/latest.svg)](https://www.npmjs.com/package/ploadin)
[![Documentation](https://img.shields.io/badge/docs-yes-brightgreen.svg)](https://github.com/JuroOravec/ploadin/tree/master/docs)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](#-contributing)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://tldrlegal.com/license/mit-license)
[![Package Size](https://img.shields.io/bundlephobia/min/ploadin)](https://bundlephobia.com/result?p=ploadin)

[![Build Status](https://travis-ci.org/JuroOravec/ploadin.svg?branch=master)](https://travis-ci.org/JuroOravec/ploadin)
![Dependencies](https://david-dm.org/JuroOravec/ploadin.svg)
[![Known Vulnerabilities](https://snyk.io/test/github/JuroOravec/ploadin/badge.svg)](https://snyk.io/test/github/JuroOravec/ploadin)
[![CodeCov](https://codecov.io/gh/JuroOravec/ploadin/branch/master/graph/badge.svg)](https://codecov.io/gh/JuroOravec/ploadin)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/JuroOravec/ploadin.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/JuroOravec/ploadin/alerts/)
[![Language grade: JavaScript](https://img.shields.io/lgtm/grade/javascript/g/JuroOravec/ploadin.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/JuroOravec/ploadin/context:javascript)
[![Maintainability](https://api.codeclimate.com/v1/badges/f1810c31ed56dfa9d125/maintainability)](https://codeclimate.com/github/JuroOravec/ploadin/maintainability)

---

<!--
One-liner explaining the purpose of the module
-->

Ploadin - Webpack **PL**ug**IN** and **LOAD**er in one. Use data from loaders in plugins and vice-versa.

#### 🏠 [Homepage](https://github.com/JuroOravec/ploadin#readme) | 🗃 [Repository](https://github.com/JuroOravec/ploadin) | 📦 [NPM](https://www.npmjs.com/package/ploadin) | 📚 [Documentation](https://github.com/JuroOravec/ploadin/tree/master/docs) | 🐛 [Issue Tracker](https://github.com/JuroOravec/ploadin/issues)

## 🪑 Table of Content

- [🧰 Features](#-features)
- [👶 Install](#-install)
- [🚀 Usage](#-usage)
- [🤖 API](#-api)
- [🔮 Background](#-background)
- [⏳ Changelog](#-changelog)
- [🛠 Developing](#-developing)
- [🏗 Roadmap](#-roadmap)
- [🤝 Contributing](#-contributing)
- [🧙 Contributors](#-contributors)
- [⭐ Show your support](#-show-your-support)
- [🐙 Community](#-community)
- [🔗 Related Projects](#-related-projects)
- [👨‍🔧 Maintainers](#-maintainers)
- [📝 License](#-license)

## 🧰 Features

<!--
A brief description of your project, what it is used for and how does life get
awesome when someone starts to use it.

- Note and briefly describe any key concepts (technical, philosophical, or both) important to the user’s understanding.
- Link to any supplementary blog posts or project main pages.
- State if it is out-of-the-box user-friendly, so it’s clear to the user.
- List its most useful/innovative/noteworthy features.
- State its goals/what problem(s) it solves.
-->

Are you developing a complex plugin that needs to be used as both a plugin and a loader? `ploadin` got you covered:

- Easily develop Webpack plugins that can access files passed through loaders.

- Use instances of `Ploadin` subclasses both as plugins and loaders.

- Plugin and loader methods share instance's state, so plugin behaviour can be modified based on data passed to loaders and vice versa.

## 👶 Install

<!--
- Getting it
- Installing It
- Configuring It
- Running it
-->

```sh
npm install ploadin
# or
yarn add ploadin
```

## 🚀 Usage

<!-- Clear, _runnable_ example of usage -->

### Subclassing

Subclass `Ploadin` to create a class that can access both plugin and loader contexts.

Following example shows how to communicate between loader and plugin. Following happens in the example:

1. `Ploadin` is subclassed to `PloadinSubclass` so the loader and plugin methods can communicate.

2. Before compilation starts (and so before loaders are run),pitch data and loader behaviour are decided based on some conditions in `apply`.

3. When `pitch` is called, the method passes data from plugin down to other `pitch` methods
   ([Webpack docs on pitch](https://webpack.js.org/api/loaders/#pitching-loader)).

4. When `loader` is called, the loader will skip its job if `ignoreLoader` was set to truthy.

5. After all is done, the state is reset in `apply` on compiler's `afterCompile` hook.

> Notice that in `loader` and `pitch` methods, `this` refers to the `Ploadin` instance and not `loaderContext` as it is in
> [Webpack loaders](https://webpack.js.org/api/loaders/#synchronous-loaders).
> The `loaderContext` is, instead, passed as the first argument, so all other arguments are shifted by one.

```js
// subclass.js
const { Ploadin, registerSubclass } = require('ploadin');

export class PloadinSubclass extends Ploadin {
  constructor() {
    this.pitchData = null;
    this.ignoreLoader = false;
    super();
  }

  // `apply` is plugin methods
  apply(compiler) {

    // Set data that pitch should pass on
    compiler.hooks.beforeCompile.tapAsync(
      'PloadinSubclass',
      (stats, callback) => {
        this.pitchData = { someData: compiler.xxx };
        this.ignoreLoader = compiler.yyy;
        callback();
      },
    );

    // Clean up
    compiler.hooks.afterCompile.tapAsync(
      'SubclassPloadin',
      (compilation, callback) => {
        this.pitchData = null
        this.ignoreLoader = false;
        callback();
      },
    );
  }

  // `loader` and `pitch` are loader methods
  loader(loaderContext, source, ...args) {
    // Skip loader action based on some conditions
    if (this.ignoreLoader) {
        return source
    }
    // Process source here otherwise...
    ...
  }

  pitch(
    loaderContext,
    remainingRequest,
    precedingRequest,
    data
  ) {
    // Pass data from plugin to pitch
    Object.assign(data, this.pitchData);
  }
}
```

<details>
  <summary>The equivalent of the above in TypeScript</summary>

```ts
// subclass.ts
import { Ploadin, registerSubclass } from 'ploadin';

export class PloadinSubclass extends Ploadin {
constructor() {
  this.pitchData = null;
  this.ignoreLoader = false;
  super();
}

// `apply` is plugin methods
apply(compiler: Compiler) {

  // Set data that pitch should pass on
  compiler.hooks.beforeCompile.tapAsync(
    'PloadinSubclass',
    (stats, callback) => {
      this.pitchData = { someData: compiler.xxx };
      this.ignoreLoader = compiler.yyy;
      callback();
    },
  );

  // Clean up
  compiler.hooks.afterCompile.tapAsync(
    'SubclassPloadin',
    (compilation, callback) => {
      this.pitchData = null
      this.ignoreLoader = false;
      callback();
    },
  );
}

// `loader` and `pitch` are loader methods
loader(loaderContext: any, source?: string, ...args: any[]) {
  // Skip loader action based on some conditions
  if (this.ignoreLoader) {
      return source
  }
  // Process source here otherwise...
  ...
}

pitch(
  loaderContext,
  remainingRequest: string,
  precedingRequest: string,
  data: any,
) {
  // Pass data from plugin to pitch
  Object.assign(data, this.pitchData);
}
}
```

</details>

### Using in Webpack

To use as plugin, pass the instance itself as a plugin.

To use as loader, pass the `asLoader` property.

```js
// webpack.config.js
const { PloadinSubclass } = require('./subclass');

const myPloadinSubclass = new PloadinSubclass();

module.exports = {
  plugins: [
      myPloadinSubclass,
  ],
  module: {
    rules: [
      {
        test: /\.js$/i, // some test
        use: [
            myPloadinSubclass.asLoader,
            ...
        ],
      },
    ],
  },
};
```

### Using multiple subclasses and instances

You can use multiple `Ploadin` subclasses, and even multiple instances of the same class, within the same config, they will not interfere.

```js
// webpack.config.js
const { PloadinSubclass } = require('./subclass1');
const { AnotherPloadinSubclass } = require('./subclass2');

const myPloadinSubclass1 = new PloadinSubclass();
const myPloadinSubclass2 = new PloadinSubclass();
const anotherPloadin1 = new AnotherPloadinSubclass();
const anotherPloadin2 = new AnotherPloadinSubclass();

module.exports = {
  plugins: [
      myPloadinSubclass1,
      myPloadinSubclass2,
      anotherPloadin1,
      anotherPloadin2
  ],
  module: {
    rules: [
      {
        test: /\.js$/i, // some test
        use: [
            myPloadinSubclass1.asLoader,
            myPloadinSubclass2.asLoader,
            anotherPloadin1.asLoader,
            anotherPloadin2.asLoader,
            ...
        ],
      },
    ],
  },
};
```

## 🤖 API

TypeDoc documentation can be [found here](https://github.com/JuroOravec/ploadin/blob/master/docs/typedoc/README.md).

---

### Ploadin

Ploadin class has following properties:

#### Ploadin.asLoader: object

Loader object to be used in webpack config.

Following methods will be called if defined:

#### Ploadin.classOptions: any

Data associated with the Ploadin class. The data returned by `classOptions` is
the same data (copy actually) of what is passed to `registerSubclass`.

#### Ploadin.apply(compiler: Compiler): void

Webpack plugin's `apply` method. See
[Writing a Webpack plugin](https://webpack.js.org/contribute/writing-a-plugin/)
for details.

#### Ploadin.loader(loaderContext: LoaderContext, content?: string, map?: string, data: any): void

Webpack loader's `loader` method. See [Webpack loaders](https://webpack.js.org/api/loaders/#synchronous-loaders)
for details.

> Note that argument signature is shifted as `loaderContext` is passed as first argument. `this`, instead, refers to `Ploadin` instance.

#### Ploadin.pitch(loaderContext: LoaderContext, remainingRequest: string, precedingRequest: string, data: any): void

Webpack loader's `pitch` method. See [Webpack loader's pitch](https://webpack.js.org/api/loaders/#pitching-loader)
for details.

> Note that argument signature is shifted as `loaderContext` is passed as first argument. `this`, instead, refers to `Ploadin` instance.

### Helpers

#### registerSubclass(subclass: Subclass, options: any): boolean

- subclass - class extending `Ploadin`
- options - any data associated with this subclass

Returns `true` if successfully registered, `false` if the class has been
registered before.

Normally, any class subclassing `Ploadin` is automatically registered with
instance-manager when a first instance is created. This is necessary so the
class and its instances can be looked up by indices.

You can register the class yourself. This enables you to optionally pass along
options associated with the given class.

One use of this is to store options passed to class factory so we can associate
the options with the dynamically-created class.

## 🔮 Background

<!-- Core Technical Concepts/Inspiration

- Potentially unfamiliar terms link to informative sources
- Why does it exist?
- Frame your project for the potential user.
- Compare/contrast your project with other, similar projects so the user knows how it is different from those projects.
- Highlight the technical concepts that your project demonstrates or supports. Keep it very brief.
- Keep it useful.
- Performs [cognitive funneling](https://github.com/noffle/art-of-readme#cognitive-funneling)
- Caveats and limitations mentioned up-front

-->

This package was prompted by the challenge of how to use and manage
dynamically created Webpack plugins that need to access both loader and plugin
contexts (similarly to how
[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin/tree/1ffc393a2e377fe0cc341cfcbc5396e07a8e4077#getting-started)
needs access to both).

Webpack passes only JSON-serializable data to loaders, so loaders don't have direct access to plugins. And if you're dealing with dynamically-created classes, correctly matching loaders with their respective plugins gets more complicated.

## ⏳ Changelog

This projects follows semantic versioning. The
[changelog can be found here](https://github.com/JuroOravec/ploadin/blob/master/CHANGELOG.md).

## 🛠 Developing

If you want to contribute to the project or forked it,
[this guide will get you up and going](https://github.com/JuroOravec/ploadin/blob/master/docs/developing.md).

## 🏗 Roadmap

This package is considered feature-complete. However, if you have ideas how it
could be improved, please be sure to share it with us by [opening an issue](#🤝-contributing).

## 🤝 Contributing

Contributions, issues and feature requests are welcome! Thank you ❤️

Feel free to dive in! See [current issues](https://github.com/JuroOravec/ploadin/issues),
[open an issue](https://github.com/JuroOravec/ploadin/issues/new), or [submit PRs](https://github.com/JuroOravec/ploadin/compare).

How to report bugs, feature requests, and how to contribute and what conventions we use is all described in the [contributing guide](https://github.com/JuroOravec/ploadin/tree/master/docs/CONTRIBUTING.md).

When contributing we follow the
[Contributor Covenant](https://contributor-covenant.org/version/1/3/0/).
See our [Code of Conduct](https://github.com/JuroOravec/ploadin/blob/master/docs/CODE_OF_CONDUCT.md).

## 🧙 Contributors

Contributions of any kind welcome. Thanks goes to these wonderful people ❤️

### Recent and Top Contributors

<!-- Hall of Fame uses 8 links (7 users + 1 stats), see https://github.com/sourcerer-io/hall-of-fame#faq -->

[![Hall of Fame Contributor 1](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/0)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/0)
[![Hall of Fame Contributor 2](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/1)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/1)
[![Hall of Fame Contributor 3](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/2)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/2)
[![Hall of Fame Contributor 4](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/3)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/3)
[![Hall of Fame Contributor 5](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/4)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/4)
[![Hall of Fame Contributor 6](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/5)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/5)
[![Hall of Fame Contributor 7](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/6)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/6)
[![Hall of Fame Contributor 8](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/images/7)](https://sourcerer.io/fame/JuroOravec/JuroOravec/ploadin/links/7)

<!-- markdownlint-disable -->

<sub><em>Generated using [Hall of Fame](https://github.com/sourcerer-io/hall-of-fame#readme).</em></sub>

<!-- markdownlint-enable -->

### All Contributors

Contribution type [emoji legend](https://allcontributors.org/docs/en/emoji-key)

<!-- ALL-CONTRIBUTORS-LIST:START - Do not remove or modify this section -->

_No additional contributors. Be the first one!_

<!-- ALL-CONTRIBUTORS-LIST:END -->

<!-- markdownlint-disable -->

<sub><em>This project follows the [all-contributors](https://github.com/all-contributors/all-contributors) specification.</em></sub>

<!-- markdownlint-enable -->

## ⭐ Show your support

Give a ⭐️if this project helped you!

## 🐙 Community

- [Stack Overflow](https://stackoverflow.com/questions/tagged/ploadin)
- [Quora](https://www.quora.com/search?q=%22ploadin%22)
- [Spectrum community](https://spectrum.chat/ploadin)

## 🔗 Related Projects

- [mini-extract-plugin](https://github.com/JuroOravec/mini-extract-plugin) -
  Generalized extensible variation of
  [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
  that can be used to create plugins to extract text from custom file types

## 👨‍🔧 Maintainers

👤 **Juro Oravec**

- Twitter: [@JuroOravec](https://twitter.com/JuroOravec)
- GitHub: [@JuroOravec](https://github.com/JuroOravec)
- LinkedIn: [@jurooravec](https://linkedin.com/in/jurooravec)
- Sourcerer: [@JuroOravec](https://sourcerer.io/JuroOravec)

## 📝 License

Copyright © 2020 [Juro Oravec](https://github.com/JuroOravec).

This project is [MIT](https://tldrlegal.com/license/mit-license) licensed.
