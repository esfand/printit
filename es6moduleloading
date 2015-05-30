# Notational Conventions

This section describes the conventions used here to describe type signatures.

A `[T]` is an array-like value (only ever used read-only in this API), i.e., one with an integer `length` and whose indexed properties from 0 to `length - 1` are of type `T`.

A type `T?` should be read as `T | undefined` -- that is, an optional value that may be `undefined`.

# Loaders

The `Loader` class is conceptually parameterized over two types:

```javascript
// Loader<ModuleAddress, ModuleSource>
```

The type parameters combine to define a protocol for communication between the steps of the pipeline hooks.

## Loader Constructor

```javascript
l = new Loader({
  realm,      // Realm?
  normalize,  // ((string, NormalizedModuleName, ModuleAddress) -> Promise<stringable>)?
  locate,     // (({ name: NormalizedModuleName, metadata: object })
              //     -> Promise<ModuleAddress>)?
  fetch,      // (({ name: NormalizedModuleName, address: ModuleAddress, metadata: object })
              //     -> Promise<ModuleSource>)?
  translate,  // (({ name: NormalizedModuleName?, address: ModuleAddress?, source: ModuleSource, metadata: object })
              //     -> Promise<string>)?
  instantiate // (({ name: NormalizedModuleName?, address: ModuleAddress?, source: ModuleSource, metadata: object })
              //     -> Promise<ModuleFactory?)?>
}) // : Loader<ModuleAddress, ModuleSource>
```

A *NormalizedModuleName* is a string obtained by coercing the resolved value of the `normalize` hook to a string.

A *ModuleFactory* is an object of the form:

```javascript
{
  deps: [string],
  execute: () -> Module
}
```

The `realm` option is a Realm object -- see the [Realm API gist](https://gist.github.com/dherman/7568885) for more details. The default value is the realm of this `Loader` constructor.

## Loader Properties

### realm

Returns the realm associated with the loader.

### global

Convenience: returns the global associated with the loader's realm.

## Loader Hooks

These are the loader hooks. They appear as instance properties when a loader is constructed via the `Loader` constructor.

### Normalization

The `normalize` hook is used to normalize context-sensitive module names that appear within a source context. Since the hook may be called multiple times and normalize to the same module name, it acts as a "fan in" at the head of the loading pipeline.

The result of the `normalize` hook is coerced into a string, which is then used as the NormalizedModuleName throughout the pipeline.

#### normalize

```javascript
// normalize : (string, NormalizedModuleName, ModuleAddress) -> Promise<stringable>
l.normalize = function(name, refererName, refererAddress) { ... }
```

### Loading pipeline

When the pipeline begins, the system constructs a new `metadata` object, which is threaded through all the steps in the pipeline. If a loader wants to carry information through the pipeline, it can either include it in the data representation of the loader's type parameters (e.g., a `ModuleSource` object could include a field that contains the `ModuleAddress` where it was loaded from), or it can attach it mutably to the `metadata` object.

The pipeline is not fed any information about the un-normalized module name, since there can be multiple requests for the same normalized name triggered from different contexts (and potentially at different times). If userland code really wants to get access to that information it can save it in a side table.

#### locate

```javascript
// locate : ({ name: NormalizedModuleName,
//             metadata: object })
//       -> Promise<ModuleAddress>
l.locate = function({ name, metadata }) { ... }
```

#### fetch

```javascript
// fetch : ({ name: NormalizedModuleName,
//            address: ModuleAddress,
//            metadata: object })
//      -> Promise<ModuleSource>
l.fetch = function({ name, address, metadata }) { ... }
```

#### translate

```javascript
// translate : ({ name: NormalizedModuleName?,
//                address: ModuleAddress?,
//                source: ModuleSource,
//                metadata: object })
//          -> Promise<string>
l.translate = function({ name, address, source, metadata }) { ... }
```

Since `loader.module()` may not provide a name or address, the `name` and `address` options may be `undefined`.

#### instantiate

```javascript
// instantiate : ({ name: NormalizedModuleName?,
//                  address: ModuleAddress?,
//                  source: ModuleSource,
//                  metadata: object })
//            -> Promise<ModuleFactory?>
l.instantiate = function({ name, address, source, metadata }) { ... }
```

Since `loader.module()` may not provide a name or address, the `name` and `address` options may be `undefined`.

## Loader Methods

These methods provide the core loader API.

### Installation

These methods can be used to install modules, and **none of them directly causes the installed modules to be executed**. (They may indirectly cause modules to be executed if they trigger custom hooks that do so.)

#### define

Installs a module in the registry from source.

*Extensible web*: This is the dynamic equivalent of a `<module>` tag in HTML.

```javascript
// define : (NormalizedModuleName, ModuleSource, { address: ModuleAddress?, metadata: object? }?) -> Promise<undefined>
p = l.define(name, defn[, options])
```

#### load

Installs a module into the registry by name or address.

*Extensible web*: This is almost the dynamic equivalent (when combined with normalization) of a declarative `import` statement, except it does not force the modules to be executed.

```javascript
// load : (NormalizedModuleName, { address: ModuleAddress? }?) -> Promise<undefined>
p = l.load(name[, options])
```

### Execution

These methods all execute code and **none of them directly causes modules to be installed**. (They may indirectly cause modules to be installed by virtue of imports in the code refering to modules that haven't yet been installed.)

#### module

Execute a top-level, anonymous module. In particular, it does not take a name and does not install the module.

*Extensible web*: This is the dynamic equivalent of an anonymous `<module>` in HTML.

```javascript
// module : (string, { address: ModuleAddress? }?) -> Promise<Module>
p = l.module(src)
```

### Installation and execution

#### import

Installs and executes a module by name, producing its instance object as the result.

*Extensible web*: This is the dynamic equivalent (when combined with normalization) of a declarative `import` statement.

```javascript
// import : (NormalizedModuleName, { address: ModuleAddress? }) -> Promise<Module>
p = l.import(name[, options])
```

This is essentially a convenience method for executing and extracting a module after loading it. That is,

```javascript
l.import(x)
```

is roughly equivalent to

```javascript
l.load(x).then(() => Promise.resolve(l.get(x)))
```

### Registry manipulation

These methods work with the module registry.

#### get

Gets the module associated with the given (normalized) key, first executing the module if it hasn't yet been executed.

```javascript
// get : (string) -> Module?
m = l.get(name)
```

#### set

```javascript
// set : (string, Module) -> undefined
l.set(name, m)
```

#### delete

```javascript
// delete : (string) -> undefined
l.delete(name)
```

#### has

```javascript
// has : (string) -> boolean
l.has(name)
```

#### keys

```javascript
// keys : Iterator<string>
for (let name of l.keys()) { ... }
```

#### values

```javascript
// values : Iterator<Module>
for (let m of l.values()) { ... }
```

#### entries

```javascript
// entries : Iterator<[string,Module]>
for (let [name, m] of l.entries()) { ... }
```

# Module Objects

A *module instance object* is a runtime instantiation of a module. They can be constructed reflectively using the `Module` constructor. A module instance object is non-extensible and its `[[Prototype]]` is **null**.

*Extensible web*: This is the dynamic equivalent of a module loaded from source.

## Module Constructor

```javascript
m = new Module(object); // object?
```

If `object` is `undefined` it defaults to an empty object. Otherwise if it's not an object a `TypeError` is thrown.

This creates a module instance object whose exported names are the *keys* (i.e., own enumerable property names) of `object` and whose export values are the result of calling `[[Get]]` on each key of `object`.

# Rationale

## Loader entry points

The `Loader` top-level API provides entry points for all steps in the pipeline that we don't deliberately want to avoid exposing.

* *NormalizedModuleName* &rarr; `locate`: `load`
* *ModuleAddress* &rarr; `fetch`: `load`
* *ModuleSource* &rarr; `translate`: `define`
* *ModuleSource* &rarr; `instantiate`: *error-prone; circumvents translation*
* *ModuleFactory* &rarr; *done*: *factories are a compatibility layer but not for general use*

## Reflection of declarative features

* `<module>`: `loader.module(...)`
* `<module id='foo'>`: `loader.define('foo', ...)`
* `<script>`: `realm.eval(...)` (perhaps deferred to ES7)
* `<iframe>`: `new Realm(...)` (perhaps deferred to ES7)
* `import from`: `loader.import(...)`
* `module from`: `loader.import(...)`
* module source: `new Module(...)`

## Deliberate omissions

  * no installing scripts (loaders only deal with modules, scripts can't be named or imported from)
  * no remote script execution (loaders deal only with modules, require fetch since `import` is language syntax; no corresponding syntax for script execution)
  * no version of `define` that executes the modules (can do this by calling `l.get()` after installation completes)
  * no version of `define` etc. that take an array (can do this with a sequence of calls since it's all async)
  * no tracking redirects; a given loader can use a mutable type for ModuleAddress and update it as it goes
