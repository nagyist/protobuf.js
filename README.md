protobuf.js [![travis][travis-image]][travis-url] [![npm][npm-image]][npm-url] [![donate][paypal-image]][paypal-url]
===========

**Protocol Buffers** are a language-neutral, platform-neutral, extensible way of serializing structured data for use in communications protocols, data storage, and more, originally designed at Google ([see](https://developers.google.com/protocol-buffers/)).

**protobuf.js** is a pure JavaScript implementation for node and the browser. It efficiently encodes plain objects and custom classes and works out of the box with .proto files.

[travis-image]: https://img.shields.io/travis/dcodeIO/protobuf.js.svg
[travis-url]: https://travis-ci.org/dcodeIO/protobuf.js
[npm-image]: https://img.shields.io/npm/v/protobufjs.svg
[npm-url]: https://npmjs.org/package/protobufjs
[paypal-image]: https://img.shields.io/badge/paypal-donate-yellow.svg
[paypal-url]: https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=dcode%40dcode.io&item_name=%3C3%20protobuf.js

Contents
--------

* [Usage](#usage)<br />
  How to include protobuf.js in your project.

* [Examples](#examples)<br />
  A few examples to get you started.

* [Module Structure](#module-structure)<br />
  A brief introduction to the structure of the exported module.

* [Documentation](#documentation)<br />
  A list of available documentation resources.

* [Command line](#command-line)<br />
  How to use the command line utility.

* [Building](#building)<br />
  How to build the library and its components yourself.

* [Performance](#performance)<br />
  A few internals and a benchmark on performance.

* [Compatibility](#compatibility)<br />
  Notes on compatibility regarding browsers and optional libraries.

Usage
---------------

### node.js

```
$> npm install protobufjs
```

```js
var protobuf = require("protobufjs");
```

### Browsers

Development:
```
<script src="//cdn.rawgit.com/dcodeIO/protobuf.js/6.0.1/dist/protobuf.js"></script>
```

Production:
```
<script src="//cdn.rawgit.com/dcodeIO/protobuf.js/6.0.1/dist/protobuf.min.js"></script>
```

The `protobuf` namespace will be available globally.

**NOTE:** Remember to replace the version tag with the exact [release](https://github.com/dcodeIO/protobuf.js/releases) your project depends upon.

Examples
--------

### Using .proto files

```protobuf
// awesome.proto
package awesomepackage;
syntax = "proto3";

message AwesomeMessage {
    string awesome_field = 1; // becomes awesomeField
}
```

```js
protobuf.load("awesome.proto", function(err, root) {
    if (err) throw err;
    
    // Obtain a message type
    var AwesomeMessage = root.lookup("awesomepackage.AwesomeMessage");

    // Create a new message
    var message = AwesomeMessage.create({ awesomeField: "AwesomeString" });

    // Encode a message (note that reflection encodes to a writer and we need to call finish)
    var buffer = AwesomeMessage.encode(message).finish();
    // ... do something with buffer

    // Or, encode a plain object (note that reflection encodes to a writer and we need to call finish)
    var buffer = AwesomeMessage.encode({ awesomeField: "AwesomeString" }).finish();
    // ... do something with buffer

    // Decode a buffer
    var message = AwesomeMessage.decode(buffer);
    // ... do something with message
});
```

You can also use promises by omitting the callback:

```js
protobuf.load("awesome.proto")
    .then(function(root) {
       ...
    });
``` 

### Using reflection only

```js
...
var Root  = protobuf.Root,
    Type  = protobuf.Type,
    Field = protobuf.Field;

var AwesomeMessage = new Type("AwesomeMessage").add(new Field(1, "awesomeField", "string"));

var root = new Root().define("awesomepackage").add(AwesomeMessage);

// Continue at "Create a new message" above
...
```

### Using custom classes

```js
...
var Prototype = protobuf.Prototype;

function AwesomeMessage(properties) {
    Prototype.call(this, properties);
}
protobuf.inherits(AwesomeMessage, root.lookup("awesomepackage.AwesomeMessage") /* or use reflection */);

var message = new AwesomeMessage({ awesomeField: "AwesomeString" });

// Encode a message (note that classes encode to a buffer directly)
var buffer = AwesomeMessage.encode(message);
// ... do something with buffer

// Or, encode a plain object (note that classes encode to a buffer directly)
var buffer = AwesomeMessage.encode({ awesomeField: "AwesomeString" });
// ... do something with buffer

// Decode a buffer
var message = AwesomeMessage.decode(buffer);
// ... do something with message
```

Custom classes are automatically populated with static `encode`, `encodeDelimited`, `decode`, `decodeDelimited` and `verify` methods and reference their reflected type via the `$type` property. Note that there are no methods (just `$type`) on instances by default as method names might conflict with field names.

Module Structure
----------------
The library exports a flat `protobuf` namespace with the following members, ordered by category:

### Parser

* **load(filename: `string|Array`, [root: `Root`], [callback: `function(err: Error, [root: Root])`]): `Promise`** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/index.js)]<br />
  Loads one or multiple .proto files into the specified root or creates a new one when omitted.

* **tokenize(source: `string`): `Object`** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/tokenize.js)]<br />
  Tokenizes the given .proto source and returns an object with useful utility functions.

* **parse(source: `string`): `Object`** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/parse.js)]<br />
  Parses the given .proto source and returns an object with the parsed contents.
  
  * **package: `string|undefined`**<br />
    The package name, if declared.

  * **imports: `Array|undefined`**<br />
    File names of imported files, if any.

  * **weakImports: `Array|undefined`**<br />
    File names of weakly imported files, if any.

  * **syntax: `string|undefined`**<br />
    Source syntax, if defined.
 
  * **root: `Root`**<br />
    The root namespace.

### Serialization

* **Writer** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/writer.js)]<br />
  Wire format writer using `Uint8Array` if available, otherwise `Array`.

* **BufferWriter** _extends **Writer**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/writer.js)]<br />
  Wire format writer using node buffers.

* **Reader** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/reader.js)]<br />
  Wire format reader using `Uint8Array` if available, otherwise `Array`.

* **BufferReader** _extends **Reader**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/reader.js)]<br />
  Wire format reader using node buffers.

* **Encoder** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/encoder.js)]<br />
  Wire format encoder using code generation on top of reflection.

* **Decoder** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/decoder.js)]<br />
  Wire format decoder using code generation on top of reflection.

* **Verifier** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/verifier.js)]<br />
  Runtime message verifier using code generation on top of reflection.

### Reflection

* **ReflectionObject** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/object.js)]<br />
  Base class of all reflection objects.

* **Namespace** _extends **ReflectionObject**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/namespace.js)]<br />
  Base class of all reflection objects containing nested objects.

* **Root** _extends **Namespace**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/root.js)]<br />
  Root namespace.

* **Type** _extends **Namespace**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/type.js)]<br />
  Reflected message type.

* **Field** _extends **ReflectionObject**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/field.js)]<br />
  Reflected message field.

* **MapField** _extends **Field**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/mapfield.js)]<br />
  Reflected message map field.

* **Enum** _extends **ReflectionObject**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/enum.js)]<br />
  Reflected enum.

* **Service** _extends **Namespace**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/service.js)]<br />
  Reflected service.

* **Method** _extends **ReflectionObject**_ [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/method.js)]<br />
  Reflected service method.

### Runtime

* **inherits(clazz: `Function`, type: `Type`, [options: `Object.<string,*>`]): `Prototype`** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/inherits.js)]<br />
  Inherits a custom class from the message prototype of the specified message type.

* **Prototype** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/prototype.js)]<br />
  Runtime message prototype ready to be extended by custom classes or generated code.

### Utility

* **util: `Object`** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/util.js)]<br />
  Utility functions.

* **common(name: `string`, json: `Object`)** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/common.js)]<br />
  Provides common type definitions.

* **types: `Object`** [[source](https://github.com/dcodeIO/protobuf.js/blob/master/src/types.js)]<br />
  Common type constants.

Documentation
-------------

* [Google's Developer Guide](https://developers.google.com/protocol-buffers/docs/overview)

* [protobuf.js API Documentation](http://dcode.io/protobuf.js/)

Command line
------------

The `pbjs` command line utility can be used to bundle and translate between .proto and .json files.

```
Consolidates imports and converts between file formats.

  -t, --target    Specifies the target format. [json, proto2, proto3]
  -p, --path      Adds a directory to the include path.
  -o, --out       Saves to a file instead of writing to stdout.

usage: pbjs [options] file1.proto file2.json ...
```

For production environments it is recommended to bundle all your .proto files to a single .json file, which reduces the number of network requests and parser invocations required:

```
$> pbjs -t json file1.proto file2.proto > bundle.json
```

Now, either include this file in your final bundle:

```js
var root = protobuf.Root.fromJSON(require("./bundle.json"));
```

or load it the usual way:

```js
protobuf.load("bundle.json", function(err, root) {
    ...
});
```

Building
--------

To build the library or its components yourself, clone it from GitHub and install the development
dependencies:

```
$> git clone https://github.com/dcodeIO/protobuf.js.git
$> cd protobuf.js
$> npm install --dev
```

Building the development and production versions with their respective source maps to `dist/`:

```
$> npm run build
```

Building the documentation to `docs/`:

```
$> npm run docs
```

Building the TypeScript definition to `types/`:

```
$> npm run types
```

Performance
-----------
The package includes a [benchmark](https://github.com/dcodeIO/protobuf.js/tree/master/bench) that tries to compare performance to native JSON as far as this is possible. On an i7-2600K running node 6.9.1 it yields:

```
benchmarking encoding performance ...

Type.encode to buffer x 402,572 ops/sec ±1.09% (90 runs sampled)
JSON.stringify to string x 342,004 ops/sec ±1.46% (82 runs sampled)
JSON.stringify to buffer x 184,468 ops/sec ±1.76% (79 runs sampled)

      Type.encode to buffer was fastest
   JSON.stringify to string was 15.4% slower
   JSON.stringify to buffer was 54.5% slower

benchmarking decoding performance ...

Type.decode from buffer x 1,170,490 ops/sec ±1.49% (88 runs sampled)
JSON.parse from string x 328,975 ops/sec ±0.90% (88 runs sampled)
JSON.parse from buffer x 298,702 ops/sec ±0.82% (89 runs sampled)

    Type.decode from buffer was fastest
     JSON.parse from string was 71.7% slower
     JSON.parse from buffer was 74.3% slower

benchmarking combined performance ...

Type to/from buffer x 218,688 ops/sec ±1.49% (90 runs sampled)
JSON to/from string x 144,634 ops/sec ±1.97% (87 runs sampled)
JSON to/from buffer x 102,350 ops/sec ±1.23% (92 runs sampled)

        Type to/from buffer was fastest
        JSON to/from string was 34.2% slower
        JSON to/from buffer was 53.1% slower
```

Note that JSON is a native binding nowadays and as such is *really* fast. So, how can protobuf.js be faster?

* The benchmark is [somewhat flawed](https://github.com/dcodeIO/protobuf.js/blob/master/bench/index.js).
* Reader and writer interfaces configure themselves according to the environment to eliminate redundant conditionals.
* Node-specific reader and writer subclasses benefit from node's buffer binding.
* Reflection has built-in code generation that builds type-specific encoders, decoders and verifiers at runtime.
* Encoders and decoders do not verify that required fields are present (with proto3 this is dead code anyway). There is a `verify` method to check this manually instead - where applicable.
* For entirely bogus values encoders intentionally rely on runtime errors to be thrown somewhere down the road.
* Quite a bit of V8-specific profiling is accountable for everything else.

Note that code generation requires `new Function(...)` (basically `eval`) support and that an equivalent but slower fallback will be used where unsupported.

Also note that as of this writing, the benchmark suite performs significantly slower on node 7.2.0 compared to 6.9.1 because moths.

Compatibility
-------------

[![Sauce Test Status](https://saucelabs.com/browser-matrix/protobuf.svg)](https://saucelabs.com/u/protobuf)

* Because the internals of this package do not rely on `google/protobuf/descriptor.proto`, options are parsed and presented literally.
* If typed arrays are not supported by the environment, plain arrays will be used instead.
* Support for pre-ES5 environments like IE8 can be achieved by [using a polyfill](https://github.com/dcodeIO/protobuf.js/blob/master/scripts/polyfill.js) and, instead of using property getters and setters on reflection objects, calling the respective functions prefixed with `get`, `set` or `is` directly (i.e. calling `Type#getFieldsById()` instead of accessing `Type#fieldsById`).
* If you need a proper way to work with 64 bit values (uint64, int64 etc.), you can install [long.js](https://github.com/dcodeIO/long.js) alongside this library. All 64 bit numbers will then be returned as a `Long` instance instead of a possibly unsafe JavaScript number ([see](https://github.com/dcodeIO/long.js)).

**License:** [Apache License, Version 2.0](http://www.apache.org/licenses/LICENSE-2.0.html), bundled external libraries may have [their own license](https://github.com/dcodeIO/protobuf.js/tree/master/lib)

[![Analytics](https://ga-beacon.appspot.com/UA-40277577-2/dcodeIO/protobuf.js)](https://github.com/igrigorik/ga-beacon)
