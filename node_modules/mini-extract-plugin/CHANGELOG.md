## [1.4.3](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.4.2...v1.4.3) (2020-05-12)

## [1.4.2](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.4.1...v1.4.2) (2020-05-12)

## [1.4.1](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.4.0...v1.4.1) (2020-05-11)


### Bug Fixes

* fix modules not being passed to moduleFilename function ([25bc438](https://github.com/JuroOravec/mini-extract-plugin/commit/25bc43835e8160ac038ed8cd547ce5f0439c1bb8))

# [1.4.0](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.3.0...v1.4.0) (2020-05-11)


### Features

* **typing:** Allow to pass MEP class to ModuleFilename types ([c82e845](https://github.com/JuroOravec/mini-extract-plugin/commit/c82e845f2be569854a18908acf58203e411873d7))

    Thus, when functions that should be used for the `moduleFilename` instance
    option can be given a type parameter, so the context argument infers
    correctly to that of the passed class.

* **typing:** Allow to pass MiniExtractPlugin class to Taps type ([9185e72](https://github.com/JuroOravec/mini-extract-plugin/commit/9185e72d704afb7d79fbcf6509e46f5c711c2509))

    And thus allow the hook function params and return types to be inferred
    correctly for the MiniExtractPlugin class that the hooks relate to. So
    correct types of Module class, Dependency
    class, etc, and correct options are all automatically inferred when the Taps
    type is provided with a MiniExtractPlugin class type

* **typing:** Pass MiniExtractPlugin type to context types ([ac05685](https://github.com/JuroOravec/mini-extract-plugin/commit/ac05685dc547ec2dc3614c43ef68704ecf217847))

    Pass MiniExtractPlugin type to context types as a first type parameter.
    This modifies the type of the `plugin`, `options`, and `classOptions`
    properties of the context objects.
    
    This allows to TypeScript to recognize subclass-specific properties /
    options. Defaults to default
MiniExtractPlugin class.

* **typing:** Pass types to class factory, classes and helpers ([cbbf8eb](https://github.com/JuroOravec/mini-extract-plugin/commit/cbbf8eb351dc2092fba718fd21483533c2e94697))

    Class factory accepts an object as a first type parameter. The object
    specifies the types of the `dependencyClass`, `dependencyTemplateClass`,
    `moduleClass`, `moduleFactoryClass` that should be
    used within the plugin. Furthermore, also the type of the constructor
    options object can be specified with `constructorOptions` property.

    Dependency class and its subclassing helper can be passed an object as a
    type parameter to specify:
    - `dependencyOptions` - the options object passed
    to the Dependency constructor.

    ModuleFactory class and its subclassing helper can be passed an object as a
    type parameter to specify:
    - `dependency` - the Dependency instance type that is passed to
    `ModuleFactory.create` method
    - `module` - the Module instance type that should be created and returned by `ModuleFactory.create` method.

    Module class and its subclassing helper can be passed an object as a type
    parameter to specify:
    - `dependency` - the Dependency instance type that is passed to Module
    constructor.

    Class factory can be passed an object as a type parameter to specify:
    - `dependencyClass` - type of the `dependencyClass` option passed to the
    factory function.
    - `dependencyTemplateClass` - type of the `dependencyTemplateClass` option
    passed to the factory function.
    - `moduleClass` - type of the `moduleClass` option passed to the factory
    function.
    - `moduleFactoryClass` - type of the `moduleFactoryClass` option passed to
    the factory function.
    - `constructorOptions` - type of the options object passed to the instance
    constructor.

    Additionally added following types to `types.util`:
    - `RequiredKeys` - Type that accepts and object type and returns a union of
    keys of required properties or never if there are
    none.

# [1.3.0](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.2.0...v1.3.0) (2020-05-09)


### Features

* **typing:** export moduleFilename function types TemplateOptions, ModuleFilenameFunction, ModuleFilename ([9e394f3](https://github.com/JuroOravec/mini-extract-plugin/commit/9e394f3ca991fe73085f42e41ac16fae9e3a9ae7))

# [1.2.0](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.1.1...v1.2.0) (2020-05-09)


### Features

* **hook:** add "initialize" hook ([8d2e907](https://github.com/JuroOravec/mini-extract-plugin/commit/8d2e9076d3b53ecf5b24aa1adaeedb12f45bf195))

## [1.1.1](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.1.0...v1.1.1) (2020-05-09)


### Bug Fixes

* **deps:** bump ploadin from 1.1.0 to 1.1.4 ([4e62ac8](https://github.com/JuroOravec/mini-extract-plugin/commit/4e62ac8811ba055780c704aca51e15e52eb312dd))

# [1.1.0](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.0.2...v1.1.0) (2020-05-09)


### Features

* **hook:** add "extracted" hook ([a069d11](https://github.com/JuroOravec/mini-extract-plugin/commit/a069d1116ca97224b1129633834de904bd4b3f4c))

## [1.0.2](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.0.1...v1.0.2) (2020-05-07)


### Bug Fixes

* **typing:** fix typing 2nd attempt ([e07acc0](https://github.com/JuroOravec/mini-extract-plugin/commit/e07acc05c9934e0fe47e2b827506b41e9e604959))

## [1.0.1](https://github.com/JuroOravec/mini-extract-plugin/compare/v1.0.0...v1.0.1) (2020-05-07)


### Bug Fixes

* **api:** fix export typos and missing exports ([1bc3e6f](https://github.com/JuroOravec/mini-extract-plugin/commit/1bc3e6f04ceb7f5c4219ad96017a2417d03eb00e))
* **typing:** fix expected types passed to subclassModuleFactory ([f829dc4](https://github.com/JuroOravec/mini-extract-plugin/commit/f829dc4447cf6d0d2ca0204d1377fccd61bcb21a))

# 1.0.0 (2020-05-06)

### Features

- initial implementation ([c8bcbc3](https://github.com/JuroOravec/mini-extract-plugin/commit/c8bcbc30cd45ebd5f13aa358c041fd7f535657e1))
