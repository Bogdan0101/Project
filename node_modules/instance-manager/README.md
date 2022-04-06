# instance-manager

[![Latest Version](https://img.shields.io/npm/v/instance-manager/latest.svg)](https://www.npmjs.com/package/instance-manager)
[![Documentation](https://img.shields.io/badge/docs-yes-brightgreen.svg)](https://github.com/JuroOravec/instance-manager/tree/master/docs)
[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](#-contributing)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://tldrlegal.com/license/mit-license)
[![Package Size](https://img.shields.io/bundlephobia/min/instance-manager)](https://bundlephobia.com/result?p=instance-manager)

[![Build Status](https://travis-ci.org/JuroOravec/instance-manager.svg?branch=master)](https://travis-ci.org/JuroOravec/instance-manager)
![Dependencies](https://david-dm.org/JuroOravec/instance-manager.svg)
[![Known Vulnerabilities](https://snyk.io/test/github/JuroOravec/instance-manager/badge.svg)](https://snyk.io/test/github/JuroOravec/instance-manager)
[![CodeCov](https://codecov.io/gh/JuroOravec/instance-manager/branch/master/graph/badge.svg)](https://codecov.io/gh/JuroOravec/instance-manager)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/JuroOravec/instance-manager.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/JuroOravec/instance-manager/alerts/)
[![Language grade: JavaScript](https://img.shields.io/lgtm/grade/javascript/g/JuroOravec/instance-manager.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/JuroOravec/instance-manager/context:javascript)
[![Maintainability](https://api.codeclimate.com/v1/badges/eaaa39ef8e79f8d2805a/maintainability)](https://codeclimate.com/github/JuroOravec/instance-manager/maintainability)

---

<!--
One-liner explaining the purpose of the module
-->

Manage references to dynamically created classes and their instances.

<!-- markdownlint-disable -->

#### 🏠 [Homepage](https://github.com/JuroOravec/instance-manager#readme) | 🗃 [Repository](https://github.com/JuroOravec/instance-manager) | 📦 [NPM](https://www.npmjs.com/package/instance-manager) | 📚 [Documentation](https://github.com/JuroOravec/instance-manager/tree/master/docs) | 🐛 [Issue Tracker](https://github.com/JuroOravec/instance-manager/issues)

<!-- markdownlint-enable -->

## 🪑 Table of Content

- [🧰 Features](#-features)
- [🔮 Background](#-background)
- [👶 Install](#-install)
- [🚀 Usage](#-usage)
- [🤖 API](#-api)
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

- Note and briefly describe any key concepts (technical, philosophical, or
  both) important to the user’s understanding.
- Link to any supplementary blog posts or project main pages.
- State if it is out-of-the-box user-friendly, so it’s clear to the user.
- List its most useful/innovative/noteworthy features.
- State its goals/what problem(s) it solves.
-->

- Useful for managing references to dynamically-created classes and their
  instances
- If you use class factory functions to create classes, you can store the class
  config along with the class in `instance-manager`
- Access classes and their instances using primitives (numbers)
- Classes and instances are weakly-references, so they will be
  garbage-collected

## 🔮 Background

<!--
Core Technical Concepts/Inspiration
- Potentially unfamiliar terms link to informative sources
- Why does it exist?
- Frame your project for the potential user.
- Compare/contrast your project with other, similar projects so the user knows
  how it is different from those projects.
- Highlight the technical concepts that your project demonstrates or supports.
  Keep it very brief.
- Keep it useful.
- Performs cognitive funneling
  https://github.com/noffle/art-of-readme#cognitive-funneling
- Caveats and limitations mentioned up-front
-->

This package is used to enable _loader_ and _plugin_ instances created from
dynamically created classes to share information in Webpack, effectively using
a single dynamically created instance to behave both as a loader and a plugin.

If you need that functionality, head over to
[ploadin](https://github.com/JuroOravec/ploadin), where the functionality has
been abstracted.

This package can be used for more cases. The problem faced can be generalized
as follows:

- There are dynamically created classes with unknown number of instances.
- You need to access a specific instance of a specific class.
- You are unable to pass the instance reference directly.

## 👶 Install

<!--
- Getting it
- Installing It
- Configuring It
- Running it
-->

```sh
npm install instance-manager
```

Then in your file

```js
const InstanceManager = require('instance-manager').default;
const im = new InstanceManager();
```

Or if using TypeScript

```ts
import InstanceManager from 'instance-manager';
const im = new InstanceManager();
```

## 🚀 Usage

<!-- Clear, _runnable_ example of usage -->

### Simple usage

```js
// -----
// Setup
// -----

// Import the package
const InstanceManager = require('instance-manager');

// Instantiate
const im = new InstanceManager();

// Class factory that creates a class based on passed options
const classFactory = (options) => {
  class MyClass {
    hello() {
      return options.message;
    }
  }
  return MyClass;
};

// Create a dynamic class
const myClassOptions = { param: 42 };
const MyDynamicClass = classFactory(myClassOptions);

// -----------------------------
// InstanceManager API - Classes
// -----------------------------

// Add classes
const arrayClassId = im.addClass(Array);

// Optionally pass an object. Useful when working with dynamically created
// classes where the object can be the parameters that were passed to the
// factory function
const myClassId = im.addClass(MyDynamicClass, myClassOptions);

// Get classes by ID
im.getClassById(arrayClassId) === Array; // true

// Or get class IDs
im.getClassId(Array) === arrayClassId; // true

// Get class options (the 2nd argument)
im.getClassOptions(MyDynamicClass) === myClassOptions; // true

// Or get class options by class ID
im.getClassOptionsById(myClassId) === myClassOptions; // true

// Remove class and all its instances along from InstanceManager
im.removeClass(Array);

// -------------------------------
// InstanceManager API - Instances
// -------------------------------

// Adds class instances.
const instanceId1 = im.addInstance(new MyDynamicClass());

// Get instance id
const anotherInstance = new MyDynamicClass();
const anotherId = im.addInstance(anotherInstance);
const sameId = im.getInstanceId(anotherInstance);
anotherId === sameId; // true

// If the class hasn't been registered before it will be added too.
const myNum = new Number('2');
const numInstanceId = im.addInstance(myNum);
im.getClassId(Number); // some number, so it is registered

// Get instance having class reference and knowing instance ID
im.getClassInstance(MyDynamicClass, anotherId) === anotherInstance; // true

// Get instance knowing only class ID and instance ID
const klass = im.getClassById(myClassId);
im.getClassInstance(klass, anotherId) === anotherInstance; // true

// Remove single instance
im.removeClassInstance(MyDynamicClass, instanceId1);

// Remove instance by reference
im.removeInstance(myNum);
```

### Advanced usage

Advanced usage is shown with TypeScript.

#### Problem

Imagine we have the following 3 files:

- `class-factory.ts` - defines a class factory.
- `create-message.ts` - dynamically creates a class and instance. Sends a
  message that is read by listener.
- `listener.ts` - reads message, needs to access an instance that created the
  message, but doesn't have a way to get the reference directly.

```ts
// class-factory.ts
export default classFactory = (data: any) => {
  class MyClass {
    makeMessage(senderContext: any) {
      // ...some behavior based on given data
    }
    doSomethingElseWithData(listenerContext: any) {
      // ...some other behavior based on given data
    }
  }
  return MyClass;
};
```

```ts
// create-message.ts
import classFactory from './class-factory';
import someData from './some-data';
import otherData from './other-data';
import messageQueue from './message-queue';

// Dynamically create your class and instance
const CustomClass = classFactory(someData);
const customInstance = new CustomClass(otherData);

const senderContext = this;
const message = customInstance.makeMessage(senderContext);

// Send signal that listener is waiting for. You can pass only strings.
// In Webpack, this is the quivalent of specifying the path to the loader
messageQueue.send(message);
```

```ts
// listener.ts
import messageQueue from './message-queue';

messageQueue.listen((listenerContext, message) => {
  // Here, we need to do something with the listenerContext, that is available
  // only in this function, but we also need to work with the data that was
  // passed to the class and instance that created this message.
  //
  // In other words, we need to call
  // customInstance.doSomethingElseWithData(listenerContext)
  //
  // But we don't have reference to the instance, so what can we do?
});
```

#### Solution

We will use the `instance-manager` to store references to each newly created
class and instance and we will attach these IDs as properties of those classes
and instance.

We will then send the IDs as strings through the message.

The listener will be able to read the IDs, and will be able to request the
correct instances from our instance of `instance-manager`.

```ts
// provider.ts
import InstanceManager from 'instance-manager';

// InstanceManager instance that will manage our dynamically created
// instances and classes
const provider = new InstanceManager<any, any>();
export default provider;
```

```ts
// class-factory.ts
import provider from './provider';

// This time, we register each created class and instance to instance-manager

export default classFactory = (data: any) => {
  class MyClass {
    constructor() {
      // Register a new instance on creation
      this._instanceId = provider.addInstance(this);
    }
    // ... the rest is same as before
  }

  // Register the newly created class on creation and keep it's ID
  MiniExtractPlugin._classId = provider.addClass(MyClass);

  return MyClass;
};
```

```ts
// create-message.ts
import classFactory from './class-factory';
import someData from './some-data';
import otherData from './other-data';
import messageQueue from './message-queue';

// Dynamically create your class and instance
const CustomClass = classFactory(someData);
const customInstance = new CustomClass(otherData);

const senderContext = this;
const message = customInstance.makeMessage(senderContext);

// This time, we can access the IDs that were generated by registering the
// class / instance with the InstanceManager
const { _classId, _instanceId } = customInstance;

// So this time, we can pass the class and instance IDs to the listener
messageQueue.send(`${message}?classId=${_classId}&instanceId=${_instanceId}`);
```

```ts
// listener.ts
import messageQueue from './message-queue';
import provider from './provider';
import { parseMessage } from './some-utils';

messageQueue.listen((listenerContext, message) => {
  // This time, we can get the class and instance IDs from the message
  const { classId, instanceId } = parseMessage(message);
  // And so we can request the correct instance from our instance of
  // instance-manager
  const klass = provider.getClassById(classId);
  const instance = provider.getClassInstance(klass, instanceId);
  // And we can successfully pass the context to the desired method
  // 🎉🎉🎉
  instance.doSomethingElseWithData(listenerContext);
});
```

## 🤖 API

Full API documentation can be [found here](https://github.com/JuroOravec/instance-manager/blob/master/docs/typedoc/README.md).

## ⏳ Changelog

This projects follows semantic versioning. The
[changelog can be found here](https://github.com/JuroOravec/instance-manager/blob/master/CHANGELOG.md).

## 🛠 Developing

If you want to contribute to the project or forked it,
[this guide will get you up and going](https://github.com/JuroOravec/instance-manager/blob/master/docs/developing.md).

## 🏗 Roadmap

This package is considered feature-complete. However, if you have ideas how it
could be improved, please be sure to share it with us by [opening an issue](#🤝-contributing).

## 🤝 Contributing

Contributions, issues and feature requests are welcome! Thank you ❤️

Feel free to dive in! See [current issues](https://github.com/JuroOravec/instance-manager/issues),
[open an issue](https://github.com/JuroOravec/instance-manager/issues/new), or
[submit PRs](https://github.com/JuroOravec/instance-manager/compare).

How to report bugs, feature requests, and how to contribute and what
conventions we use is all described in the
[contributing guide](https://github.com/JuroOravec/instance-manager/tree/master/docs/CONTRIBUTING.md).

When contributing we follow the
[Contributor Covenant](https://contributor-covenant.org/version/1/3/0/).
See our [Code of Conduct](https://github.com/JuroOravec/instance-manager/blob/master/docs/CODE_OF_CONDUCT.md).

## 🧙 Contributors

Contributions of any kind welcome. Thanks goes to these wonderful people ❤️

### Recent and Top Contributors

<!--
Hall of Fame uses 8 links (7 users + 1 stats),
see https://github.com/sourcerer-io/hall-of-fame#faq
-->

[![Hall of Fame Contributor 1](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/0)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/0)
[![Hall of Fame Contributor 2](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/1)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/1)
[![Hall of Fame Contributor 3](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/2)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/2)
[![Hall of Fame Contributor 4](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/3)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/3)
[![Hall of Fame Contributor 5](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/4)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/4)
[![Hall of Fame Contributor 6](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/5)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/5)
[![Hall of Fame Contributor 7](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/6)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/6)
[![Hall of Fame Contributor 8](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/images/7)](https://sourcerer.io/fame/JuroOravec/JuroOravec/instance-manager/links/7)

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

Give a ⭐️ if this project helped you!

## 🐙 Community

- [Stack Overflow](https://stackoverflow.com/questions/tagged/instance-manager)
- [Quora](https://www.quora.com/search?q=%22instance-manager%22)
- [Spectrum community](https://spectrum.chat/instance-manager)

## 🔗 Related Projects

- [ploadin](https://github.com/JuroOravec/ploadin) - Webpack plugin and loader
  in one
- [mini-extract-plugin](https://github.com/JuroOravec/mini-extract-plugin) -
  Generalized extensible variation of
  [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
  that can be used to create plugins to extract text from custom file types

## 👨‍🔧 Maintainers

👤 **Juro Oravec**

- Twitter: [@JuroOravec](https://twitter.com/JuroOravec)
- GitHub: [@JuroOravec](https://github.com/JuroOravec)
- LinkedIn: [@jurooravec](https://linkedin.com/in/jurooravec)
- Sourcerer: [@JuroOravec](https://sourcerer.io/jurooravec)

## 📝 License

Copyright © 2020 [Juro Oravec](https://github.com/JuroOravec).

This project is [MIT](https://tldrlegal.com/license/mit-license) licensed.
