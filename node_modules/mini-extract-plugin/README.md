# mini-extract-plugin

[![Latest Version](https://img.shields.io/npm/v/mini-extract-plugin/latest.svg)](https://www.npmjs.com/package/mini-extract-plugin)
[![Documentation](https://img.shields.io/badge/docs-yes-brightgreen.svg)](https://github.com/JuroOravec/mini-extract-plugin/tree/master/docs)
[![contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](#-contributing)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://tldrlegal.com/license/mit-license)
[![Package Size](https://img.shields.io/bundlephobia/min/mini-extract-plugin)](https://bundlephobia.com/result?p=mini-extract-plugin)

[![Build Status](https://travis-ci.org/JuroOravec/mini-extract-plugin.svg?branch=master)](https://travis-ci.org/JuroOravec/mini-extract-plugin)
![Dependencies](https://david-dm.org/JuroOravec/mini-extract-plugin.svg)
[![Known Vulnerabilities](https://snyk.io/test/github/JuroOravec/mini-extract-plugin/badge.svg)](https://snyk.io/test/github/JuroOravec/mini-extract-plugin)
[![codecov](https://codecov.io/gh/JuroOravec/mini-extract-plugin/branch/master/graph/badge.svg)](https://codecov.io/gh/JuroOravec/mini-extract-plugin)
[![Total alerts](https://img.shields.io/lgtm/alerts/g/JuroOravec/mini-extract-plugin.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/JuroOravec/mini-extract-plugin/alerts/)
[![Language grade: JavaScript](https://img.shields.io/lgtm/grade/javascript/g/JuroOravec/mini-extract-plugin.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/JuroOravec/mini-extract-plugin/context:javascript)
[![Maintainability](https://api.codeclimate.com/v1/badges/58410cfd7c7351aab086/maintainability)](https://codeclimate.com/github/JuroOravec/mini-extract-plugin/maintainability)

---

<!--
One-liner explaining the purpose of the module
-->

Generalized and hookable
[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin).
Extract any format, process it your way.

#### 🏠 [Homepage](https://github.com/JuroOravec/mini-extract-plugin#readme) | 🗃 [Repository](https://github.com/JuroOravec/mini-extract-plugin) | 📦 [NPM](https://www.npmjs.com/package/mini-extract-plugin) | 📚 [Documentation](https://github.com/JuroOravec/mini-extract-plugin/tree/master/docs) | 🐛 [Issue Tracker](https://github.com/JuroOravec/mini-extract-plugin/issues)

## 🪑 Table of Content

- [🧰 Features](#-features)
- [👶 Install](#-install)
- [🚀 Usage](#-usage)
  - [Minimal setup](#minimal-setup)
  - [Subclassing](#subclassing)
  - [Instance options](#instance-options)
  - [Helpers](#helpers)
  - [Typing](#typing)
  - [Debugging](#debugging)
- [🔮 Background](#-background)
- [🤖 API](#-api)
  - [Options](#options)
  - [Hooks order](#hooks-order)
  - [Hooks](#hooks)
  - [Contexts](#contexts)
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

- Exposes 18 hooks that enable you to:

  - Override how the modules are parsed when passed to the loader
  - Override how the modules are merged into a common file
  - Override what content is returned instead of the content was extracted.
  - Define if and how the modules should should be split
  - Hook into the compiler / compilation.
  - Extend initialization

- Based on
  [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
  v0.9.0, this package recycles the same logic, but allows you override the
  format-specific logic and more.

- See how the original mini-css-extract-plugin has been
  [reimplemented using this package](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin).

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

## 👶 Install

```sh
npm install mini-extract-plugin
```

## 🚀 Usage

<!-- Clear, _runnable_ example of usage -->

### Minimal setup

For a minimal setup, just import the plugin and give it a unique type name by
which the plugin will be known. The instances of the created class can be used
in Webpack.

```ts
// Default import is a class factory
import mep from 'mini-extract-plugin';
// or const mep = require('mini-extract-plugin').default

// Minimal config
const MyMiniExtractPlugin =  mep({
  type: 'my-custom-type'
});

// Create instance to be used in the config.
// Class constructor optionally accepts an options object
const myMEP = new MyMiniExtractPlugin({
  ...
});

// webpack config
const config = {
  ...
  module: {
    rules: [
      {
        // We want all encountered files named "*.some.json"
        // to be extracted into separate files.
        //
        // The files could be e.g. i18n files, or some config
        // files that are defined in multiple places but we want
        // a single file that combines all matched files.
        resourceQuery: /\.some\.json$/,
        use: [
          // Register the plugin as a loader
          // `asLoader` is an object so we have to use it in the
          // `use` array
          myMEP.asLoader,
        ],
        ...
      }
    ],
  },
  plugins: [
    // Register the plugin as a plugin.
    myMEP,
    ...
  ]
}
```

However, this example above is not terribly useful since we haven't specified
any hooks. That means, for example, that found JSONs will be concatenated into
as single file as strings (e.g. `"{}" + "{}"`), but that won't yield a valid
JSON. So to make the new plugin class to work properly, we need to speficy more
than just the type.

```ts
import mep from 'mini-extract-plugin';
// const mep = require('mini-extract-plugin').default

const MyMiniExtractPlugin = mep({
  type: 'my-custom-type',
  displayName: `My Super Awesome Extract Plugin`,
  hooks: [
    // Tap sync function `compilationHookFn` into `compilation`
    // hook.
    // Maybe we want to tap into the Webpack's Compilation
    // instance
    {
      name: 'compilation',
      type: 'tap',
      hooks: [compilationHookFn],
    },
    // Tap sync function `mergeHookFn` into `merge` hook.
    // Here we will override how the modules should be
    // concatenated (instead of string concat, we join JSONs
    // as objects)
    { name: 'merge', type: 'tap', hooks: [mergeHookFn] },
  ],
});

// Create instance to be used in the config.
const myMEP = new MyMiniExtractPlugin();
...
```

This is better! We have specified how the modules should be merged and so when
we use this plugin in the Webpack now, the end result will be a proper JSON.

This way, you can override only the parts of the process that needs to be
modified. You can also tap multiple functions into any single hook. And where
available, you can also tap asynchronously. Some hooks are "waterfall", meaning
that result from one function is passed to another. Other are regular hooks
where the tapped functions don't interact. See
[`tapable`](https://github.com/webpack/tapable)
for details on how hooks work.

For details on what hooks are available, their signatures, whether they're sync
or async, etc, see [Hooks](#hooks).

You may have noticed that we've also specified a `displayName` property.
Additional options allow you to:

- override what classes should be used for modules, dependencies, and other
  Webpack entities (useful if you want to pass custom data along with the
  modules).
- override what options can be passed to the class constructor when creating a
  plugin instance (by default,
  [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
  options are used)
- override the name of the created class, and how it is displayed.

[See full description of the class options here](###options).

#### Examples

- [mini-css-extract-plugin fixture](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin) -
  Test re-implementation of
  [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
- [mini-i18n-extract-plugin](https://github.com/JuroOravec/mini-i18n-extract-plugin) -
  Extract i18n files (JSON / YAML) and split the files by entrypoints and language locales.

### Subclassing

#### Class factory

Here's an example how MiniExtractPlugin can be subclassed taken from the [mini-css-extract-plugin replimenetation](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin)

```ts
import miniExtractPluginFactory, { types } from 'mini-extract-plugin';
import Module from './module';
import ModuleFactory from './module-factory';
import Dependency from './dependency';
import DependencyTemplate from './dependency-template';
import hooks from './hooks';
import { type, typeReadable } from './config';

const MiniExtractPluginClass = miniExtractPluginFactory<{
  dependencyClass: types.DependencyClass<Dependency>;
  moduleClass: types.ModuleClass<Module>;
  moduleFactoryClass: typeof ModuleFactory;
}>({
  type,
  displayName: `My Mini ${typeReadable} Extract Plugin`,
  moduleFactoryClass: ModuleFactory,
  dependencyClass: Dependency,
  dependencyTemplateClass: DependencyTemplate,
  hooks: [
    { name: 'compilation', type: 'tap', hooks: [hooks.compilation!] },
    { name: 'merge', type: 'tap', hooks: [hooks.merge!] },
  ],
});

export default MiniExtractPluginClass;
```

The factory function is passed:

- identifiers `type` and optional `displayName`.
- custom subclasses `dependencyClass`, `moduleClass`, and
  `dependencyTemplateClass`.
- [`compilation`](#compilation) and [`merge`](#merge) [hooks](#hooks).

Factory function was also given an object as a type parameter. This object specifies which types should be used for classes and options, and enable type inferrence for subclasses with custom classes and options.

Full list of type options with their defaults:

```ts
const MyMiniExtractPlugin = miniExtractPluginFactory<{
  // These reflect the types of the classes that we pass to the
  // class factory as options.
  dependencyClass?: DependencyClass;
  dependencyTemplateClass?: DependencyTemplateClass;
  moduleClass?: ModuleClass;
  moduleFactoryClass?: ModuleFactoryClass;
  // Type of the options object passed to constructor on instantiations.
  constructorOptions?: { [key: string]: any };
} = {}
>(...)
```

#### Options classes

Classes that can be passed to the factory function can be found at the root of
the export. Subclassing is as simple as:

```ts
import {
  Dependency,
  DependencyTemplate,
  Module,
  ModuleFactory,
} from 'mini-extract-plugin';

class DependencySubclass extends Dependency {}
class DependencyTemplateSubclass extends DependencyTemplate {}
class ModuleSubclass extends Module {}
class ModuleFactorySubclass extends ModuleFactory {}
```

If you use TypeScript and want to override the types these subclass use in methods / constructor, you can pass type arguments. This is useful e.g. if your subclass adds properties, and you want TypeScript to recognize those properties.

All type parameters that can be passed to classes + their defaults:

```ts
import {
  subclassDependency,
  subclassDependencyTemplate,
  subclassModule,
  subclassModuleFactory,
} from 'mini-extract-plugin';

class DependencySubclass extends Dependency<{
  // Options object passed to the Dependency constructor
  dependencyOptions: DependencyOptions;
}> {}

// subclassDependencyTemplate has no type parameters
class DependencyTemplateSubclass extends DependencyTemplate {}

class ModuleSubclass extends Module<{
  // Dependency class whose instance is passed to the Module
  // constructor
  dependency: Dependency;
}> {}

class ModuleFactorySubclass extends ModuleFactory<{
  // Dependency class that is passed to ModuleFactory.create
  dependency: Dependency;
  // Module class whose instance is created in ModuleFactory.create
  // from Dependency
  module: Module;
}> {}
```

#### Subclassing helpers

If you need to subclass any of the above but don't need to override the behaviour,
you can use helper subclassing functions.

Each of them accepts options

```ts
import {
  subclassDependency,
  subclassDependencyTemplate,
  subclassModule,
  subclassModuleFactory,
} from 'mini-extract-plugin';

// `type` is the same `type` argument passed to class factory
const DependencySubclass = subclassDependency({ type: 'custom-plugin' });
const DependencyTemplateSubclass = subclassDependencyTemplate({
  type: 'custom-plugin',
});
const ModuleSubclass = subclassModule({ type: 'custom-plugin' });
const ModuleFactorySubclass = subclassModuleFactory({
  type: 'custom-plugin',
  moduleClass: MyCustomModule, // Optionally define module class
});
```

If you use TypeScript and want to override the classes these subclass use, you can pass type arguments. The type arguments passed to subclass helpers are same as to the classes.

Given the example from above, if you want to module type created by `ModuleFactorySubclass` to match `MyCustomModule`, you can do following:

```ts
const ModuleFactorySubclass = subclassModuleFactory<{
    module: Module;
  }({
  type: 'custom-plugin',
  moduleClass: MyCustomModule,
});
```

All type parameters that can be passed to helper functions + their defaults:

```ts
import {
  subclassDependency,
  subclassDependencyTemplate,
  subclassModule,
  subclassModuleFactory,
} from 'mini-extract-plugin';

const DependencySubclass = subclassDependency<{
  // Options object passed to the Dependency constructor
  dependencyOptions: DependencyOptions;
}>(...);

// subclassDependencyTemplate has no type parameters
const DependencyTemplateSubclass = subclassDependencyTemplate(...);

const ModuleSubclass = subclassModule<{
  // Dependency class whose instance is passed to the Module
  // constructor
  dependency: Dependency;
}>(...);

const ModuleFactorySubclass = subclassModuleFactory<{
  // Dependency class that is passed to ModuleFactory.create
  dependency: Dependency;
  // Module class whose instance is created in ModuleFactory.create
  // from Dependency
  module: Module;
}>(...);
```

See how classes are extended in the
[re-implementation of mini-css-extract-plugin](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin).

### Instance options

By default, instances of the subclassed MiniExtractPlugin accept an options object with the same options as [mini-css-extract-plugin v.0.9.0](https://github.com/webpack-contrib/mini-css-extract-plugin/tree/v0.9.0).

> :warning: Some options, while having same name and fulfilling same task,
> accept different values than those of
> [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin/tree/v0.9.0). These are:

#### moduleFilename

The function signature is changed to:

`(`[`RenderContext`](#RenderContext)`, TemplateOptions,`
[`Module[]`](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js)`)`
`=> string;`

| Argument                                                                                                   | Description                                                                                                                                                                                                              |
| ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`RenderContext`](#RenderContext)                                                                          | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
| `TemplateOptions`                                                                                          | Object with info on the chunk that is being rendered:<br>`{` <br> &nbsp;&nbsp;`chunk: Chunk,` <br> &nbsp; &nbsp;`hash: string,` <br> &nbsp;&nbsp; `contentHashType: string` <br> `}`                                     |
| [`Module`](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js) | [Module](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js) that are being rendered. Modules are of the class as specified in the [plugin's class options](#options).       |

### Helpers

The package also exposes a `util` export, which contains utility modules used
for working with the hooks or objects passed to hook functions:

- `util.module` includes helper functions for working with modules.
- `util.subclass` includes helper functions for subclassing the classes that
  can be passed to the class factory. Use these functions if you need a
  subclass but don't care about implementation.

### Typing

This project is written in TypeScript and the typings are included under the
`types` import.

- `types`, the root, includes interfaces related to subclassing
  (MiniExtractPlugin, class factory options, etc.)
- `types.context` includes interfaces for the [contexts](#Contexts) passed to
  tapped functions
- `types.hook` includes types related to Hooks (Hook overrides, types of
  recognized hooks, etc.)
- `types.webpack` is a shim for Webpack, it includes types that either are not
  exposed in Webpack v4, or which have modified interfaces in the extraction
  process, so the provided types reflect the actual interface.
- `types.util` includes helper types which may or may not be useful when
  working with the hooks.

What you will most likely want is to have the hook functions types inferred.
You can use the `types.hook.Taps` interface, which is an object of
`{ [hookName]: hookFunction }`.
If you want to define only some hooks, use `types.hook.PartialTaps`.

Both types accept MiniExtractPlugin interface as a type parameter. Use it to have correct types for arguments and return functions. The passed interface affects the inferred types of context objects, classes passed to MiniExtractPlugin (module, dependency, moduleFactory, ...).

```ts
import { types } from 'mini-extract-plugin';

import { IMyMiniExtractPlugin } from './types'

const hooks: types.hook.PartialTaps<IMyMiniExtractPlugin> = {
  // Arguments of `dependency` are inferred thanks to
  // `PartialTaps`
  dependency: (context, {exports: exported}) => {
    const { childCompilation, classOptions } = context;
    ...
};
```

### Debugging

This project uses [debug](https://www.npmjs.com/package/debug). To show debug
logs, activate debug for `mini-extract-plugin`.

CLI example:

```sh
DEBUG=mini-extract-plugin node path/to/my/mini-extract-plugin-project
```

## 🔮 Background

[mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
is great because it allows you to have modularized definitions, which are then
merged or split as necessary for you during the build.

This is important for bundle optimization, but also for maintainability, as you
can keep the information where it makes sense, without making a tradeoff in
logistics.

When working with Vue, I was hoping to manage other auxiliary file types (i18n
and documentation, to be specific) in a similar manner - modularized definition
but processed and emitted as separate files during build.

There's (understandably) not as much support for other file formats as there is
for CSS. But since other formats could benefit from same action,generalizing
the process was in order (and thus encouraging modularized approach for more
formats).

## 🤖 API

TypeDoc documentation can be
[found here](https://github.com/JuroOravec/mini-extract-plugin/blob/master/docs/typedoc/README.md).

### Options

```ts
interface ClassOptions {
  type: string;
  pluginName?: string;
  displayName?: string;
  className?: string;
  moduleType?: string;
  pluginOptionsSchema?: any;
  loaderOptionsSchema?: any;
  dependencyTemplateClass?: DependencyTemplateClass;
  dependencyClass?: DependencyClass;
  moduleFactoryClass?: ModuleFactoryClass;
  moduleClass?: ModuleClass;
  hooks?: Overrides;
}
```

The class factory accepts an object with following options:

- `type` - _(Required)_ Namespace used by this class to distinguish it, its
  instances, and webpack resources (e.g. modules and dependencies) from other
  MiniExtractPlugin classes.
  - E.g. MiniCssExtractPlugin uses `css`, MiniI18nExtractPlugin uses `i18n`
- `pluginName` - Name of plugin used in identifiers. Prefer package-like
  (kebab-case) format.
  - Default: `mini-${type}-extract-plugin`.
- `displayName` - String to be used when printing the class in logging messages
  or similar.
  - Default: `Mini ${type} Extract Plugin` where `type` is capitalized.
- `className` - Name of the plugin class that is shown when calling either
  `class.toString()` or `class.name`.
  - Default: `Mini${type}ExtractPlugin` where `type` is capitalized.
- `moduleType` - Identifier used to find modules processed by the class.
  - Default: `${type}/mini-extract`.
- `pluginOptionsSchema` - JSON schema used to validate options passed to
  constructor and used in loader methods.
  - Default: MiniCssExtractPlugin's
    [plugin options schema](https://github.com/webpack-contrib/mini-css-extract-plugin/blob/1ffc393a2e377fe0cc341cfcbc5396e07a8e4077/src/plugin-options.json)
- `loaderOptionsSchema` - JSON schema used to validate options passed to
  constructor and used in plugin methods.
  - Default: MiniCssExtractPlugin's
    [plugin options schema](https://github.com/webpack-contrib/mini-css-extract-plugin/blob/1ffc393a2e377fe0cc341cfcbc5396e07a8e4077/src/loader-options.json)
- `dependencyTemplateClass` - Class that implements Webpack's
  [DependencyTemplate](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/DependencyTemplate.js)
  interface.
  - The instance of `dependencyTemplateClass` must have a method `apply`.
  - Default: Default is empty implementation.
    ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/models/dependency-template.ts)).
- `dependencyClass` - Class that implements Webpack's
  [Dependency](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Dependency.js)
  interface.
  - Dependencies are used among other to pass data to Modules. Data passed to
    Dependency constructor can be overriden in `dependency` hook.
  - If providing ypur own Dependency class, be sure to subclass the default
    implementation to ensure the plugin works as expected. See the
    [re-implementation of MiniCssExtractPlugin](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin/dependency.ts)
    for an example of how the Dependency class can be extended.
  - Default: Default implementation ensures the MiniExtractPlugin works
    correctly. It stores information on MiniExtractPlugin type, identifier,
    context, and content
    ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/models/dependency.ts)).
- `moduleFactoryClass` - Class that implements Webpack's
  [Module Factory](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/ModuleFactory.js)
  interface.
  - The instance of `moduleFactoryClass` must have a method `create` that is
    called with data and callback and must call the callback with either an
    error as first argument or
    [Module](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js)
    instance as second argument. The data contains an array-wrapped dependency of
    class `dependencyClass`.
  - Default: Default implementation passed the dependency to Module constructor
    ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/models/module-factory.ts)).
    If `moduleClass` is specified, the created Module will be of class
    `moduleClass`. Otherwise [Webpack's Module class](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js)
    is used.
- `moduleClass` - Class that implements Webpack's
  [Module](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js)
  interface.
  - This class used only with the default `moduleFactoryClass`
    implementation. If you specify `moduleFactoryClass`, this option is
    ignored.
  - Module constructor accepts a dependency of class `dependencyClass`.
  - If providing your own Module class, be sure to subclass the default
    implementation to ensure the plugin works as expected. See the
    [re-implementation of MiniCssExtractPlugin](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin/module.ts)
    for an example of how the Module class can be extended.
  - Default: Default implementation ensures the MiniExtractPlugin works
    correctly, and it updates identifiers and hashes
    ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/models/module.ts)).
- `hooks` - Array of objects specifying which Tapable hooks should be tapped
  with what functions, and how the hook should behave (sync/async). See
  [Hooks](#hooks) for the list of available hooks.

  - Each object is expected to implement the hook
    [Override](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/types/hooks.ts)
    interface:

    | Property | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | Type         | Default |
    | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------ | ------- |
    | `name`   | Name of the hook to be tapped.                                                                                                                                                                                                                                                                                                                                                                                                                                                 | `string`     | -       |
    | `type`   | How should be the functions applied to the hook. These are the methods defined by [tapable](https://github.com/webpack/tapable)<br><br>Options: `'tap'` \| `'tapAsync'` \| `'tapPromise'` \| `'intercept'`<br><br> Note: `tapAsync` and `tapPromise` are available only for async hooks. See [tapable documentation](https://github.com/webpack/tapable) for details.                                                                                                          | `string`     | -       |
    | `hooks`  | Array of functions that should be applied to the hook.<br><br>Functions are expected to conform to the hook signature they are tapping to. For details on using sync- / promise- / callback-style functions, see [tapable documentation](https://github.com/webpack/tapable).<br><br>See how the hooks are defined in [MiniCssExtractPlugin re-implementation](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin/module.ts). | `Function[]` | `[]`    |

### Hooks order

Hooks are called in following order:

- _Plugin initialization_
  - [`initialize`](#initialize)
- _`Plugin.apply` is called_
  - [`compiler`](#compiler)
  - [`compilation`](#compilation)
- _Modules passed to loader: `Loader.pitch` is called and modules are processed
  to dependencies_
  - [`pitch`](#pitch)
  - [`childCompiler`](#childCompiler)
  - [`source`](#source)
  - [`childCompilation`](#childCompilation)
  - [`dependency`](#dependency)
  - [`extracted`](#extracted)
- _If non-entrypoint chunks are emitted: Render and merge chunks' modules_
  - [`beforeRenderChunk`](#beforeRenderChunk)
  - [`renderChunk`](#renderChunk)
  - [`afterRenderChunk`](#afterRenderChunk)
  - _If using default `renderChunk`, specify module merging_
    - [`beforeMerge`](#beforeMerge)
    - [`merge`](#merge)
    - [`afterMerge`](#afterMerge)
- _Render and merge entrypoint's modules_
  - [`beforeRenderMain`](#beforeRenderMain)
  - [`renderMain`](#renderMain)
  - [`afterRenderMain`](#afterRenderMain)
  - _If using default `renderMain`, specify module merging_
    - [`beforeMerge`](#beforeMerge)
    - [`merge`](#merge)
    - [`afterMerge`](#afterMerge)

### Hooks

The available hooks in alphabetical order are
([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/test/fixtures/mini-css-extract-plugin/module.ts)):

#### afterMerge

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Source`](https://github.com/webpack/webpack-sources)`) => void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Hook called after Modules were merged into a single Source that will be
  emitted into a file.

  Use this hook if you want to modify the resulting Source without overriding
  the merging process itself, or if you want to trigger some behaviour after
  the [`merge`](#merge) step has finished.

- Default: No behaviour.

  | Argument                                               | Description                                                                                                                                                                                                              |
  | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderContext`](#RenderContext)                      | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
  | [`Source`](https://github.com/webpack/webpack-sources) | Instance of Webpack's [`Source`](https://github.com/webpack/webpack-sources) (value returned by [`merge`](#merge)).                                                                                                      |

#### afterRenderChunk

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`RenderManifestEntry[]`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)`)`
  `=> void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Hook called after the list of
  [`RenderManifestEntries`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)
  has been prepared for the generation of chunk (non-entrypoint) files.

  Use this hook if you want to modify the resulting list of
  [`RenderManifestEntries`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)
  without overriding the [`renderChunk`](#renderChunk) process itself, or if
  you want to trigger some behaviour after the [`renderChunk`](#renderChunk)
  step has finished.

- Default: No behaviour.

  | Argument                                                                                                                      | Description                                                                                                                                                                                                                                                       |
  | ----------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`RenderContext`](#RenderContext)                                                                                             | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details.                                          |
  | [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) | List of Webpack's [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) objects that will be used to render and create output files. (values processed by [`renderChunk`](#renderChunk)). |

#### afterRenderMain

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`RenderManifestEntry[]`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)`)`
  `=> void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Hook called after the list of
  [`RenderManifestEntries`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)
  has been prepared for the generation of main (entry) files.

  Use this hook if you want to modify the resulting list of
  [`RenderManifestEntries`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)
  without overriding the [`renderMain`](#renderMain) process itself, or if
  you want to trigger some behaviour after the [`renderMain`](#renderMain)
  step has finished.

- Default: No behaviour.

  | Argument                                                                                                                      | Description                                                                                                                                                                                                                                                     |
  | ----------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`RenderContext`](#RenderContext)                                                                                             | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details.                                        |
  | [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) | List of Webpack's [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) objects that will be used to render and create output files. (values processed by [`renderMain`](#renderMain)). |

#### beforeMerge

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Module[]`](#options)`) =>`
  [`Module[]`](#options)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Hook called when merging multiple Modules into a single Source that will be
  emitted into a file.

  Use this hook if you want to modify the list of modules without overriding
  the merging process itself.

- Default: No modifications done.

| Argument                          | Description                                                                                                                                                                                                              |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`RenderContext`](#RenderContext) | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
| [`Modules`](#options)             | List of [`Options.moduleClass`](#options) modules.                                                                                                                                                                       |

| Returns               | Description                                                  |
| --------------------- | ------------------------------------------------------------ |
| [`Modules`](#options) | Processed list of [`Options.moduleClass`](#options) modules. |

#### beforeRenderChunk

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Module[]`](#options)`) =>`
  [`Module[] | Module[][]`](#options)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Hook called when Webpack is generating a chunk (non-entrypoint file) from
  the given set of modules.

  Use this hook if you want to modify the list of modules or if you want to
  split the list of modules into multiple chunks without overriding the
  rendering process itself.

  This hook is called only if the extracted Dependencies from
  [`dependency`](#dependency) hook were split into chunks by Webpack.

- Default: No modifications done.

  | Argument                          | Description                                                                                                                                                                                                              |
  | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderContext`](#RenderContext) | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
  | [`Modules`](#options)             | List of [`Options.moduleClass`](#options) modules.                                                                                                                                                                       |

  | Returns         | Description                                                                                                                                                                                  |
  | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | `ModulesGroups` | List or list of lists of [`Options.moduleClass`](#options) modules. If a list of lists is returned, these are interpreted as module groups. Each module group will emit separate chunk file. |

#### beforeRenderMain

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Module[]`](#options)`) =>`
  [`Module[] | Module[][]`](#options)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Hook called when Webpack is generating a main (entry) file from
  the given set of modules.

  Use this hook if you want to modify the list of modules or if you want to
  split the list of modules into multiple groups without overriding the
  rendering process itself.

- Default: No modifications done.

  | Argument                          | Description                                                                                                                                                                                                              |
  | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderContext`](#RenderContext) | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
  | [`Modules`](#options)             | List of [`Options.moduleClass`](#options) modules.                                                                                                                                                                       |

  | Returns                     | Description                                                                                                                                                                            |
  | --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`ModulesGroups`](#options) | List or list of lists of [`Options.moduleClass`](#options) modules. If a list of lists is returned, these are interpreted as module groups. Each module group will emit separate file. |

#### childCompilation

- Signature: `(`[`PitchCompilationContext`](#PitchCompilationContext)`) => void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Hook called when the compilation of [child Compiler](#childCompiler) is run.

  Use this hook if you need to modify the child Compiler's Compilation or if
  you want to access child Compiler's Compilation's hooks yourself.

- Default: No behaviour.

  | Argument                                              | Description                                                                                                                                                                                                                                                                            |
  | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`PitchCompilationContext`](#PitchCompilationContext) | Context available in loader's [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) function on child Compiler's [`thisCompilation`](https://webpack.js.org/api/compiler-hooks/#thiscompilation) hook. See [`PitchCompilationContext`](#PitchCompilationContext) for details. |

#### childCompiler

- Signature: `(`[`PitchCompilerContext`](#PitchCompilerContext)`) => void`

- Hook: [`AsyncParallelHook`](https://github.com/webpack/tapable)

- Hook called after a child Compiler was set up in loader's `pitch` method.
  Child Compiler is used to evaluate the modules passed to the loader, and the
  resulting content will be extracted.

  Use this hook if you need to modify the child Compiler before other hooks are
  tapped, or if you want to access child Compiler's hooks yourself.

- Default: No behaviour.

  | Argument                                        | Description                                                                                                                                                                                                    |
  | ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`PitchCompilerContext`](#PitchCompilerContext) | Context available in loader's [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) function on creation of child Compiler instance. See [`PitchCompilerContext`](#PitchCompilerContext) for details. |

#### compiler

- Signature: `(`[`CompilerContext`](#CompilerContext)`) => void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Hook called at the beginning of plugin's
  [`apply`](https://webpack.js.org/contribute/writing-a-plugin/)
  method. Use if you need to access Compiler hooks or if you need to set things
  up at the beginning of the process.

- Default: No behaviour.

  | Argument                              | Description                                                                                                                                                 |
  | ------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`CompilerContext`](#CompilerContext) | Context available in Plugin's [`apply`](https://webpack.js.org/contribute/writing-a-plugin/) method. See [`CompilerContext`](#CompilerContext) for details. |

#### compilation

- Signature: `(`[`CompilationContext`](#CompilationContext)`) => void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Hook called at the beginning of Compiler's
  [`thisCompilation`](https://webpack.js.org/api/compiler-hooks/#thiscompilation)
  hook. Use if you need to access Compilation hooks.

- Default: No behaviour.

  | Argument                          | Description                                                                                                                                                                        |
  | --------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`RenderContext`](#RenderContext) | Context available on Compiler's [`thisCompilation`](https://webpack.js.org/api/compiler-hooks/#thiscompilation) hook. See [`CompilationContext`](#CompilationContext) for details. |

#### dependency

- Signature: `(`[`PitchCompilationContext`](#PitchCompilationContext)`,`
  [`LoaderModuleContext`](#LoaderModuleContext)`) =>`
  [`DependencyOptions[]`](#Options)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Process the data that was obtained from evaluating the source code of the
  Module that was passed to loader's `pitch` method.

  The data should be processed to objects that can be passed to
  [`dependencyClass`](#options), which will be used by Webpack to create the
  extracted files.

  Use this hook if you need to modify the data from Module before it is passed
  to the [`dependencyClass`](#options), or if need to split/merge the evaluated
  data.

- Default:
  [MiniCssExtractPlugins](https://github.com/webpack-contrib/mini-css-extract-plugin)-derived
  behaviour. Exported data is interpreted as an arrray of [ID, content]
  ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/methods/pitch.ts)).

| Argument                                              | Description                                                                                                                                                                                                                                                                            |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`PitchCompilationContext`](#PitchCompilationContext) | Context available in loader's [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) function on child Compiler's [`thisCompilation`](https://webpack.js.org/api/compiler-hooks/#thiscompilation) hook. See [`PitchCompilationContext`](#PitchCompilationContext) for details. |
| [`LoaderModuleContext`](#LoaderModuleContext)         | Data generated by evaluating the source code of the Module that triggered the MiniExtractPlugin's loader. See [`LoaderModuleContext`](#LoaderModuleContext).                                                                                                                           |
|                                                       |

| Returns                         | Description                                                                                                                                                                                                       |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`DependencyOptions`](#options) | List of options objects to be passed to the [`dependencyClass`](#options) to create Webpack's [Dependencies](https://github.com/webpack/webpack/blob/a4ad311b909d9a4d68069b3c2283b80613eaa5e7/lib/Dependency.js). |

#### extracted

- Signature: `(`[`PitchCompilationContext`](#PitchCompilationContext)`,`
  `string) => string`

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Modify the source string of the module from which content has been extracted
  before it is passed to Webpack.

  Use this hook if you need to modify the string so it can conforms to a loader
  / parser that it will be passed to next.

- Default: Inserts comment `// extracted by` _`plugin-name`_ plus HMR compat.

| Argument                                              | Description                                                                                                                                                                                                                                                                            |
| ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`PitchCompilationContext`](#PitchCompilationContext) | Context available in loader's [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) function on child Compiler's [`thisCompilation`](https://webpack.js.org/api/compiler-hooks/#thiscompilation) hook. See [`PitchCompilationContext`](#PitchCompilationContext) for details. |
| `remainingSource`                                     | String that will be returned to Webpack as the source of the module from which data has been extracted.                                                                                                                                                                                |
|                                                       |

| Returns  | Description                                                                                             |
| -------- | ------------------------------------------------------------------------------------------------------- |
| `string` | String that will be returned to Webpack as the source of the module from which data has been extracted. |

#### initialize

- Signature: `(MiniExtractPlugin, object) => void`

- Hook: [`SyncHook`](https://github.com/webpack/tapable)

- Modify the MiniExtractPlugin instance during initialization (called from
  constructor). This hook is called after other initialization logic is done.

  Use this hook if you need to extend the class with custom methods /
  properties, or to set default options values.

- Default: No behaviour.

| Argument          | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `instance`        | `MiniExtractPlugin` instance that is being constructed.      |
| `instanceOptions` | Options object with which the instance is being constructed. |

#### merge

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Module[]`](#options)`) =>`
  [`Source`](https://github.com/webpack/webpack-sources)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Hook called when merging multiple Modules into a single Source that will be
  emitted into a file.

  Use this hook if you want to override how Modules are merged.

- Default: Join contents with newline (`\n`).

| Argument                          | Description                                                                                                                                                                                                              |
| --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [`RenderContext`](#RenderContext) | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
| [`Modules`](#options)             | List of [`Options.moduleClass`](#options) modules (values processed by [`beforeMerge`](#beforeMerge)).                                                                                                                   |

| Returns                                                | Description                                                                                                               |
| ------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------- |
| [`Source`](https://github.com/webpack/webpack-sources) | Instance of Webpack's [`Source`](<[`Source`](https://github.com/webpack/webpack-sources)>) with content from the modules. |

#### pitch

- Signature: `(`[`PitchContext`](#PitchContext)`) => void`

- Hook: [`AsyncParallelHook`](https://github.com/webpack/tapable)

- Hook called at the beginning of loader's
  [`pitch`](https://webpack.js.org/contribute/writing-a-plugin/)
  method. Use if you need to set up the loader environment.

- Default: No behaviour.

| Argument                        | Description                                                                                                                                             |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`PitchContext`](#PitchContext) | Context available in loader's [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) function. See [`PitchContext`](#PitchContext) for details. |

#### renderChunk

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Module[][]`](#options)`) =>`
  [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Hook called when Webpack is generating a chunk (non-entrypoint file) from
  the given set of module groups. Each module group should yield a separate
  file. The hook should return a list of
  [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)
  objects which specify the metadata of the to-be-generated file(s).

  Use this hook if you want to override how the module groups are rendered and
  processed for the chunk generation.

  This hook is called only if the extracted Dependencies from
  [`dependency`](#dependency) hook were split into chunks by Webpack.

  - Default:
    [MiniCssExtractPlugins](https://github.com/webpack-contrib/mini-css-extract-plugin)-derived
    behaviour
    ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/methods/apply.ts)).

  | Argument                          | Description                                                                                                                                                                                                              |
  | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderContext`](#RenderContext) | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
  | [`moduleGroups`](#options)        | List of lists of [`Options.moduleClass`](#options) modules (values processed by [`beforeRenderChunk`](#beforeRenderChunk)).                                                                                              |

  | Returns                                                                                                                       | Description                                                                                                                                                                                                        |
  | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) | List of Webpack's [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) objects that will be used to render and create output chunk files. |

#### renderMain

- Signature: `(`[`RenderContext`](#RenderContext)`,`
  [`Module[][]`](#options)`) =>`
  [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)

- Hook: [`SyncWaterfallHook`](https://github.com/webpack/tapable)

- Hook called when Webpack is generating a main (entrypoint) file from
  the given set of module groups. Each module group should yield a separate
  file. The hook should return a list of
  [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57)
  objects which specify the metadata of the to-be-generated file(s).

  Use this hook if you want to override how the module groups are rendered and
  processed for the entry file generation.

  - Default:
    [MiniCssExtractPlugins](https://github.com/webpack-contrib/mini-css-extract-plugin)-derived
    behaviour
    ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/methods/apply.ts)).

  | Argument                          | Description                                                                                                                                                                                                              |
  | --------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderContext`](#RenderContext) | Context available on Compilation's [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. See [`RenderContext`](#RenderContext) for details. |
  | [`moduleGroups`](#options)        | List of lists of [`Options.moduleClass`](#options) modules (values processed by [`beforeRenderMain`](#beforeRenderMain)).                                                                                                |

  | Returns                                                                                                                       | Description                                                                                                                                                                                                  |
  | ----------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
  | [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) | List of Webpack's [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) objects that will be used to render and create output files. |

#### source

- Signature: `(`[`PitchCompilationContext`](#PitchCompilationContext)`) => string`

- Hook: [`AsyncSeriesWaterfallHook`](https://github.com/webpack/tapable)

- Get source code from a Module that was passed to loader's `pitch` method. Use
  this hook if you need to modify the source code before it is evaluated.

- Default: No modifications done.

  | Argument                                              | Description                                                                                                                                                                                                                                                                            |
  | ----------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
  | [`PitchCompilationContext`](#PitchCompilationContext) | Context available in loader's [`pitch`](https://webpack.js.org/api/loaders/#pitching-loader) function on child Compiler's [`thisCompilation`](https://webpack.js.org/api/compiler-hooks/#thiscompilation) hook. See [`PitchCompilationContext`](#PitchCompilationContext) for details. |

  | Returns            | Description                                                                                     |
  | ------------------ | ----------------------------------------------------------------------------------------------- |
  | `ModuleSourceCode` | String representation of the [`Module's`](#options) content that was intercepted by the loader. |

### Contexts

Hooks can be tapped to modify the extraction process at different stages.
Therefore, also the information available is different. That's why different
hooks expose different "contexts", or objects with contextual information
available at the point in time of the call.

If you're using TypeScript, the type of the `MiniExtractPlugin` that is
accessible from the context objects (together with its `options` and
`classOptions`) can be overriden by passing your custom subclass as the first
type parameter to the context type.

```ts
import { types } from 'mini-extract-plugin';
let context: types.context.CompilationContext<MyMiniExtractPlugin>;
```

Here is the list of used contexts ([see source file](https://github.com/JuroOravec/mini-extract-plugin/tree/master/src/types/context.ts)):

#### CompilerContext

| Property       | Description                                                                                                                                                                                    | Type                                                                         |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `plugin`       | Instance of `MiniExtractPlugin` where the process occurs                                                                                                                                       | `MiniExtractPlugin`                                                          |
| `classOptions` | Class options used to create the `MiniExtractPlugin` class that was used to create this instance. Shorthand for `plugin.classOptions`. See [Options](#options).                                | [`Options`](#options)                                                        |
| `options`      | Options passed to the `MiniExtractPlugin` instance. Shorthand for `plugin.options`. Options object is defined by the `pluginOptionsSchema` and `loaderOptionsSchema` [class option](#options). | `object`                                                                     |
| `compiler`     | Webpack's [Compiler](https://webpack.js.org/api/compiler-hooks/) instance exposed to the plugin's `apply` method.                                                                              | [`Compiler`](https://github.com/webpack/webpack/blob/master/lib/Compiler.js) |

#### CompilationContext

Same as [CompilerContext](#CompilerContext) plus following:

| Property      | Description                                                                                                                                                                                                                                       | Type                                                                               |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `compilation` | Webpack's [Compilation](https://webpack.js.org/api/compilation-object/) instance exposed by tapping to Compiler's [`thisCompilation`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) hook. | [`Compilation`](https://github.com/webpack/webpack/blob/master/lib/Compilation.js) |

#### RenderContext

Same as [CompilationContext](#CompilationContext) plus following:

| Property        | Description                                                                                                                                                                                                                                                                                                                                                                                                                                           | Type                                                                                                                            |
| --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `renderEntries` | List of Webpack's [RenderManifestEntry](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/Template.js#L57) objects to be rendered, exposed by tapping to [MainTemplate's](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js) [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook. | [`RenderManifestEntry`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/Template.js#L57)   |
| `renderOptions` | Webpack's [RenderManifestOptions](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/Template.js#L43) exposed by tapping to [MainTemplate's](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js) [`renderManifest`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/MainTemplate.js#L44) hook.                               | [`RenderManifestOptions`](https://github.com/webpack/webpack/blob/f3ad9752234d416c64330d3b50d6d6bd5abdd0e4/lib/Template.js#L43) |  |

<details><summary>Example of <code>RenderManifestOptions</code> object</summary>
<pre>
{
  chunk: Chunk {
    id: 'entry2',
    ids: [Array],
    debugId: 1047,
    name: 'entry2',
    preventIntegration: false,
    entryModule: [NormalModule],
    _modules: [SortableSet [Set]],
    filenameTemplate: undefined,
    _groups: [SortableSet [Set]],
    files: [],
    rendered: false,
    hash: '61a4b63e502b1ca58699200b35ff1691',
    contentHash: [Object: null prototype],
    renderedHash: '61a4b63e502b1ca58699',
    chunkReason: undefined,
    extraAsync: false,
    removedModules: undefined
  },
  hash: '9d88bf494be3f1082cdb',
  fullHash: '9d88bf494be3f1082cdbb4388c176e06',
  outputOptions: {
    path: '/Users/path/to/project/dist',
    filename: '[name].js',
    chunkFilename: '[name].js',
    webassemblyModuleFilename: '[modulehash].module.wasm',
    library: '',
    hotUpdateFunction: 'webpackHotUpdate',
    jsonpFunction: 'webpackJsonp',
    chunkCallbackName: 'webpackChunk',
    globalObject: 'window',
    devtoolNamespace: '',
    libraryTarget: 'var',
    pathinfo: true,
    sourceMapFilename: '[file].map[query]',
    hotUpdateChunkFilename: '[id].[hash].hot-update.js',
    hotUpdateMainFilename: '[hash].hot-update.json',
    crossOriginLoading: false,
    jsonpScriptType: false,
    chunkLoadTimeout: 120000,
    hashFunction: 'md4',
    hashDigest: 'hex',
    hashDigestLength: 20,
    devtoolLineToLine: false,
    strictModuleExceptionHandling: false
  },
  moduleTemplates: { javascript: [ModuleTemplate], webassembly: [ModuleTemplate] },
  dependencyTemplates: Map(34) {
    'hash' => '',
    [Function: CssDependency] => CssDependencyTemplate {},
    [Function] => ConstDependencyTemplate {},
    ...
  }
}
</pre></details>

#### PitchContext

| Property           | Description                                                                                                                                                                                    | Type                                                                      |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `plugin`           | Instance of `MiniExtractPlugin` where the process occurs                                                                                                                                       | `MiniExtractPlugin`                                                       |
| `classOptions`     | Class options used to create the `MiniExtractPlugin` class that was used to create this instance. Shorthand for `plugin.classOptions`. See [Options](#options).                                | [`Options`](#options)                                                     |
| `options`          | Options passed to the `MiniExtractPlugin` instance. Shorthand for `plugin.options`. Options object is defined by the `pluginOptionsSchema` and `loaderOptionsSchema` [class option](#options). | `object`                                                                  |
| `loaderContext`    | Webpack's [Loader Context](https://webpack.js.org/api/loaders/#the-loader-context).                                                                                                            | [`LoaderContext`](https://webpack.js.org/api/loaders/#the-loader-context) |
| `remainingRequest` | Request part ahead of the pitch. A string passed to `pitch` as the first argument. See Webpack's [Loader API](https://webpack.js.org/api/loaders/#pitching-loader).                            | `string`                                                                  |
| `precedingRequest` | Request part past the pitch. A string passed to `pitch` as the second argument. See Webpack's [Loader API](https://webpack.js.org/api/loaders/#pitching-loader).                               | `string`                                                                  |
| `data`             | Data object to passed to `pitch` as the third argument. See Webpack's [Loader API](https://webpack.js.org/api/loaders/#pitching-loader).                                                       | `object`                                                                  |

#### PitchCompilerContext

Same as [PitchContext](#PitchContext) plus the following:

| Property        | Description                                                                                                              | Type                                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- |
| `childCompiler` | Child instance of Webpack's [Compiler](https://webpack.js.org/api/compiler-hooks/) used to render the extracted modules. | [`Compiler`](https://github.com/webpack/webpack/blob/master/lib/Compiler.js) |

#### PitchCompilationContext

Same as [PitchCompilerContext](#PitchCompilerContext) plus the following:

| Property           | Description                                                                                                                                                                                                                                             | Type                                                                               |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| `childCompilation` | Webpack's [Compilation](https://webpack.js.org/api/compilation-object/) instance exposed by tapping to child Compiler's [`thisCompilation`](https://github.com/webpack/webpack/blob/ee94677a7729fac726b7100256986a7a7db8c53e/lib/Template.js#L57) hook. | [`Compilation`](https://github.com/webpack/webpack/blob/master/lib/Compilation.js) |

#### LoaderModuleContext

Data generated by evaluating the source code of the
[Module](https://github.com/webpack/webpack/blob/96142f829f7df4bfe6aa5095b1aed647de50edfe/lib/Module.js)
that triggered the MiniExtractPlugin's loader. The data is used to create
[Dependencies](https://github.com/webpack/webpack/blob/a4ad311b909d9a4d68069b3c2283b80613eaa5e7/lib/Dependency.js)
that will be used to emit files with extracted content.

| Property  | Description                                                                            | Type     |
| --------- | -------------------------------------------------------------------------------------- | -------- |
| `source`  | String representation of the source code (value returned by [`source`](#source) hook). | `string` |
| `locals`  | Local variables from the evaluated source code.                                        | `any`    |
| `exports` | Exported variables from the evaluated source code.                                     | `any`    |

## ⏳ Changelog

This projects follows semantic versioning. The
[changelog can be found here](https://github.com/JuroOravec/mini-extract-plugin/blob/master/CHANGELOG.md).

## 🛠 Developing

If you want to contribute to the project or forked it,
[this guide will get you up and going](https://github.com/JuroOravec/mini-extract-plugin/blob/master/docs/developing.md).

## 🏗 Roadmap

There is no explicit roadmap for this project. However, if you have ideas how it
could be improved, please be sure to share it with us by [opening an issue](#🤝-contributing).

## 🤝 Contributing

Contributions, issues and feature requests are welcome! Thank you ❤️

Feel free to dive in! See [current issues](https://github.com/JuroOravec/mini-extract-plugin/issues),
[open an issue](https://github.com/JuroOravec/mini-extract-plugin/issues/new), or [submit PRs](https://github.com/JuroOravec/mini-extract-plugin/compare).

How to report bugs, feature requests, and how to contribute and what
conventions we use is all described in the
[contributing guide](https://github.com/JuroOravec/mini-extract-plugin/tree/master/docs/CONTRIBUTING.md).

When contributing we follow the
[Contributor Covenant](https://contributor-covenant.org/version/1/3/0/).
See our [Code of Conduct](https://github.com/JuroOravec/mini-extract-plugin/blob/master/docs/CODE_OF_CONDUCT.md).

## 🧙 Contributors

Contributions of any kind welcome. Thanks goes to these wonderful people ❤️

### Recent and Top Contributors

<!-- Hall of Fame uses 8 links (7 users + 1 stats), see https://github.com/sourcerer-io/hall-of-fame#faq -->

[![Hall of Fame Contributor 1](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/0)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/0)
[![Hall of Fame Contributor 2](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/1)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/1)
[![Hall of Fame Contributor 3](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/2)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/2)
[![Hall of Fame Contributor 4](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/3)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/3)
[![Hall of Fame Contributor 5](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/4)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/4)
[![Hall of Fame Contributor 6](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/5)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/5)
[![Hall of Fame Contributor 7](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/6)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/6)
[![Hall of Fame Contributor 8](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/images/7)](https://sourcerer.io/fame/JuroOravec/JuroOravec/mini-extract-plugin/links/7)

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

- [Stack Overflow](https://stackoverflow.com/questions/tagged/mini-extract-plugin)
- [Quora](https://www.quora.com/search?q=%22mini-extract-plugin%22)
- [Spectrum community](https://spectrum.chat/mini-extract-plugin)

## 🔗 Related Projects

This project is based on [mini-css-extract-plugin](https://github.com/webpack-contrib/mini-css-extract-plugin)
and the amazing work of [Tobias Koppers](https://github.com/sokra) (@sokra)
and other [Webpack maintainers](https://github.com/orgs/webpack/people).

## 👨‍🔧 Maintainers

👤 **Juro Oravec**

- Twitter: [@JuroOravec](https://twitter.com/JuroOravec)
- GitHub: [@JuroOravec](https://github.com/JuroOravec)
- LinkedIn: [@jurooravec](https://linkedin.com/in/jurooravec)
- Sourcerer: [@JuroOravec](https://sourcerer.io/JuroOravec)

## 📝 License

Copyright © 2020 [Juro Oravec](https://github.com/JuroOravec).

This project is [MIT](https://tldrlegal.com/license/mit-license) licensed.
