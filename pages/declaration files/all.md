# Declaration Files

This guide is designed to teach you how to write a high-quality TypeScript Declaration File.

In this guide, we'll assume basic familiarity with the TypeScript language.
If you haven't already, you should read the [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/basic-types.html)
  to familiarize yourself with basic concepts, especially types and namespaces.

## Sections

The guide is broken down into the following sections.

### Library Structures

The [Library Structures](./Library Structures.md) guide helps you understand common library formats and how to write a correct declaration file for each format.
If you're editing an existing file, you probably don't need to read this section.
Authors of new declaration files must read this section to properly understand how the format of the library influences the writing of the declaration file.

### By Example

Many times, we are faced with writing a declaration file when we only have examples of the underlying library to guide us.
The [By Example](./By Example.md) section shows many common API patterns and how to write declarations for each of them.
This guide is aimed at the TypeScript novice who may not yet be familiar with every language construct in TypeScript.

### "Do"s and "Don't"s

Many common mistakes in declaration files can be easily avoided.
The [Do's and Don'ts](./Do's and Don'ts.md) section identifies common errors,
  describes how to detect them,
  and how to fix them.
Everyone should read this section to help themselves avoid common mistakes.

### Deep Dive

For seasoned authors interested in the underlying mechanics of how declaration files work,
  the [Deep Dive](./Deep Dive.md) section explains many advanced concepts in declaration writing,
  and shows how to leverage these concepts to create cleaner and more intuitive declaration files.

### Templates

In [Templates](./Templates.md) you'll find a number of declaration files that serve as a useful starting point
  when writing a new file.
Refer to the documentation in [Library Structures](./Library Structures.md) to figure out which template file to use.

### Publish to npm

The [Publishing](./Publishing.md) section explains how to publish your declaration files to an npm package, and shows how to manage your dependent packages.

### Find and Install Declaration Files

For JavaScript library users, the [Consumption](./Consumption.md) section offers a few simple steps to locate and install corresponding declaration files.

# Library Structures

## Overview

Broadly speaking, the way you *structure* your declaration file depends on how the library is consumed.
There are many ways of offering a library for consumption in JavaScript, and you'll need to write your declaration file to match it.
This guide covers how to identify common library patterns, and how to write declaration files which correspond to that pattern.

Each type of major library structuring pattern has a corresponding file in the [Templates](./Templates.md) section.
You can start with these templates to help you get going faster.

## Identifying Kinds of Libraries

First, we'll review the kinds of libraries TypeScript declaration files can represent.
We'll briefly show how each kind of library is *used*, how it is *written*, and list some example libraries from the real world.

Identifying the structure of a library is the first step in writing its declaration file.
We'll give hints on how to identify structure both based on its *usage* and its *code*.
Depending on the library's documentation and organization, one might be easier than the other.
We recommend using whichever is more comfortable to you.

### Global Libraries

A *global* library is one that can be accessed from the global scope (i.e. without using any form of `import`).
Many libraries simply expose one or more global variables for use.
For example, if you were using [jQuery](https://jquery.com/), the `$` variable can be used by simply referring to it:

```ts
$(() => { console.log('hello!'); } );
```

You'll usually see guidance in the documentation of a global library of how to use the library in an HTML script tag:

```html
<script src="http://a.great.cdn.for/someLib.js"></script>
```

Today, most popular globally-accessible libraries are actually written as UMD libraries (see below).
UMD library documentation is hard to distinguish from global library documentation.
Before writing a global declaration file, make sure the library isn't actually UMD.

#### Identifying a Global Library from Code

Global library code is usually extremely simple.
A global "Hello, world" library might look like this:

```js
function createGreeting(s) {
    return "Hello, " + s;
}
```

or like this:

```js
window.createGreeting = function(s) {
    return "Hello, " + s;
}
```

When looking at the code of a global library, you'll usually see:

* Top-level `var` statements or `function` declarations
* One or more assignments to `window.someName`
* Assumptions that DOM primitives like `document` or `window` exist

You *won't* see:

* Checks for, or usage of, module loaders like `require` or `define`
* CommonJS/Node.js-style imports of the form `var fs = require("fs");`
* Calls to `define(...)`
* Documentation describing how to `require` or import the library

#### Examples of Global Libraries

Because it's usually easy to turn a global library into a UMD library, very few popular libraries are still written in the global style.
However, libraries that are small and require the DOM (or have *no* dependencies) may still be global.

#### Global Library Template

The template file [`global.d.ts`](./templates/global.d.ts.md) defines an example library `myLib`.
Be sure to read the ["Preventing Name Conflicts" footnote](#preventing-name-conflicts).

### Modular Libraries

Some libraries only work in a module loader environment.
For example, because `express` only works in Node.js and must be loaded using the CommonJS `require` function.

ECMAScript 2015 (also known as ES2015, ECMAScript 6, and ES6), CommonJS, and RequireJS have similar notions of *importing* a *module*.
In JavaScript CommonJS (Node.js), for example, you would write

```ts
var fs = require("fs");
```

In TypeScript or ES6, the `import` keyword serves the same purpose:

```ts
import fs = require("fs");
```

You'll typically see modular libraries include one of these lines in their documentation:

```js
var someLib = require('someLib');
```

or

```ts
define(..., ['someLib'], function(someLib) {

});
```

As with global modules, you might see these examples in the documentation of a UMD module, so be sure to check the code or documentation.

#### Identifying a Module Library from Code

Modular libraries will typically have at least some of the following:

* Unconditional calls to `require` or `define`
* Declarations like `import * as a from 'b';` or `export c;`
* Assignments to `exports` or `module.exports`

They will rarely have:

* Assignments to properties of `window` or `global`

#### Examples of Modular Libraries

Many popular Node.js libraries are in the module family, such as [`express`](http://expressjs.com/), [`gulp`](http://gulpjs.com/), and [`request`](https://github.com/request/request).

### *UMD*

A *UMD* module is one that can *either* be used as module (through an import), or as a global (when run in an environment without a module loader).
Many popular libraries, such as [Moment.js](http://momentjs.com/), are written this way.
For example, in Node.js or using RequireJS, you would write:

```ts
import moment = require("moment");
console.log(moment.format());
```

whereas in a vanilla browser environment you would write:

```ts
console.log(moment.format());
```

#### Identifying a UMD library

[UMD modules](https://github.com/umdjs/umd) check for the existence of a module loader environment.
This is an easy-to-spot pattern that looks something like this:

```js
(function (root, factory) {
    if (typeof define === "function" && define.amd) {
        define(["libName"], factory);
    } else if (typeof module === "object" && module.exports) {
        module.exports = factory(require("libName"));
    } else {
        root.returnExports = factory(root.libName);
    }
}(this, function (b) {
```

If you see tests for `typeof define`, `typeof window`, or `typeof module` in the code of a library, especially at the top of the file, it's almost always a UMD library.

Documentation for UMD libraries will also often demonstrate a "Using in Node.js" example showing `require`,
  and a "Using in the browser" example showing using a `<script>` tag to load the script.

#### Examples of UMD libraries

Most popular libraries are now available as UMD packages.
Examples include [jQuery](https://jquery.com/), [Moment.js](http://momentjs.com/), [lodash](https://lodash.com/), and many more.

#### Template

There are three templates available for modules,
  [`module.d.ts`](./templates/module.d.ts.md), [`module-class.d.ts`](./templates/module-class.d.ts.md) and [`module-function.d.ts`](./templates/module-function.d.ts.md).

Use [`module-function.d.ts`](./templates/module-function.d.ts.md) if your module can be *called* like a function:

```ts
var x = require("foo");
// Note: calling 'x' as a function
var y = x(42);
```

Be sure to read the [footnote "The Impact of ES6 on Module Call Signatures"](#the-impact-of-es6-on-module-plugins)

Use [`module-class.d.ts`](./templates/module-class.d.ts.md) if your module can be *constructed* using `new`:

```ts
var x = require("bar");
// Note: using 'new' operator on the imported variable
var y = new x("hello");
```

The same [footnote](#the-impact-of-es6-on-module-plugins) applies to these modules.

If your module is not callable or constructable, use the [`module.d.ts`](./templates/module.d.ts.md) file.

### *Module Plugin* or *UMD Plugin*

A *module plugin* changes the shape of another module (either UMD or module).
For example, in Moment.js, `moment-range` adds a new `range` method to the `moment` object.

For the purposes of writing a declaration file, you'll write the same code whether the module being changed is a plain module or UMD module.

#### Template

Use the [`module-plugin.d.ts`](./templates/module-plugin.d.ts.md) template.

### *Global Plugin*

A *global plugin* is global code that changes the shape of some global.
As with *global-modifying modules*, these raise the possibility of runtime conflict.

For example, some libraries add new functions to `Array.prototype` or `String.prototype`.

#### Identifying global plugins

Global plugins are generally easy to identify from their documentation.

You'll see examples that look like this:

```ts
var x = "hello, world";
// Creates new methods on built-in types
console.log(x.startsWithHello());

var y = [1, 2, 3];
// Creates new methods on built-in types
console.log(y.reverseAndSort());
```

#### Template

Use the [`global-plugin.d.ts`](./templates/global-plugin.d.ts.md) template.

### *Global-modifying Modules*

A *global-modifying module* alters existing values in the global scope when they are imported.
For example, there might exist a library which adds new members to `String.prototype` when imported.
This pattern is somewhat dangerous due to the possibility of runtime conflicts,
  but we can still write a declaration file for it.

#### Identifying global-modifying modules

Global-modifying modules are generally easy to identify from their documentation.
In general, they're similar to global plugins, but need a `require` call to activate their effects.

You might see documentation like this:

```ts
// 'require' call that doesn't use its return value
var unused = require("magic-string-time");
/* or */
require("magic-string-time");

var x = "hello, world";
// Creates new methods on built-in types
console.log(x.startsWithHello());

var y = [1, 2, 3];
// Creates new methods on built-in types
console.log(y.reverseAndSort());
```

#### Template

Use the [`global-modifying-module.d.ts`](./templates/global-modifying-module.d.ts.md) template.

## Consuming Dependencies

There are several kinds of dependencies you might have.

### Dependencies on Global Libraries

If your library depends on a global library, use a `/// <reference types="..." />` directive:

```ts
/// <reference types="someLib" />

function getThing(): someLib.thing;
```

### Dependencies on Modules

If your library depends on a module, use an `import` statement:

```ts
import * as moment from "moment";

function getThing(): moment;
```

### Dependencies on UMD libraries

#### From a Global Library

If your global library depends on a UMD module, use a `/// <reference types` directive:

```ts
/// <reference types="moment" />

function getThing(): moment;
```

#### From a Module or UMD Library

If your module or UMD library depends on a UMD library, use an `import` statement:

```ts
import * as someLib from 'someLib';
```

Do *not* use a `/// <reference` directive to declare a dependency to a UMD library!

## Footnotes

### Preventing Name Conflicts

Note that it's possible to define many types in the global scope when writing a global declaration file.
We strongly discourage this as it leads to possible unresolvable name conflicts when many declaration files are in a project.

A simple rule to follow is to only declare types *namespaced* by whatever global variable the library defines.
For example, if the library defines the global value 'cats', you should write

```ts
declare namespace cats {
    interface KittySettings { }
}
```

But *not*

```ts
// at top-level
interface CatsKittySettings { }
```

This guidance also ensures that the library can be transitioned to UMD without breaking declaration file users.

### The Impact of ES6 on Module Plugins

Some plugins add or modify top-level exports on existing modules.
While this is legal in CommonJS and other loaders, ES6 modules are considered immutable and this pattern will not be possible.
Because TypeScript is loader-agnostic, there is no compile-time enforcement of this policy, but developers intending to transition to an ES6 module loader should be aware of this.

### The Impact of ES6 on Module Call Signatures

Many popular libraries, such as Express, expose themselves as a callable function when imported.
For example, the typical Express usage looks like this:

```ts
import exp = require("express");
var app = exp();
```

In ES6 module loaders, the top-level object (here imported as `exp`) can only have properties;
  the top-level module object is *never* callable.
The most common solution here is to define a `default` export for a callable/constructable object;
  some module loader shims will automatically detect this situation and replace the top-level object with the `default` export.
  
# By Example

## Introduction

The purpose of this guide is to teach you how to write a high-quality definition file.
This guide is structured by showing documentation for some API, along with sample usage of that API,
  and explaining how to write the corresponding declaration.

These examples are ordered in approximately increasing order of complexity.

* [Global Variables](#global-variables)
* [Global Functions](#global-functions)
* [Objects with Properties](#objects-with-properties)
* [Overloaded Function](#overloaded-functions)
* [Reusable Types (Interfaces)](#reusable-types-interfaces)
* [Reusable Types (Type Aliases)](#reusable-types-type-aliases)
* [Organizing Types](#organizing-types)
* [Classes](#classes)

## The Examples

### Global Variables

*Documentation*

> The global variable `foo` contains the number of widgets present.

*Code*

```ts
console.log("Half the number of widgets is " + (foo / 2));
```

*Declaration*

Use `declare var` to declare variables.
If the variable is read-only, you can use `declare const`.
You can also use `declare let` if the variable is block-scoped.

```ts
/** The number of widgets present */
declare var foo: number;
```

### Global Functions

*Documentation*

> You can call the function `greet` with a string to show a greeting to the user.

*Code*

```ts
greet("hello, world");
```

*Declaration*

Use `declare function` to declare functions.

```ts
declare function greet(greeting: string): void;
```

### Objects with Properties

*Documentation*

> The global variable `myLib` has a function `makeGreeting` for creating greetings,
> and a property `numberOfGreetings` indicating the number of greetings made so far.

*Code*

```ts
let result = myLib.makeGreeting("hello, world");
console.log("The computed greeting is:" + result);

let count = myLib.numberOfGreetings;
```

*Declaration*

Use `declare namespace` to describe types or values accessed by dotted notation.

```ts
declare namespace myLib {
    function makeGreeting(s: string): string;
    let numberOfGreetings: number;
}
```

### Overloaded Functions

*Documentation*

The `getWidget` function accepts a number and returns a Widget, or accepts a string and returns a Widget array.

*Code*

```ts
let x: Widget = getWidget(43);

let arr: Widget[] = getWidget("all of them");
```

*Declaration*

```ts
declare function getWidget(n: number): Widget;
declare function getWidget(s: string): Widget[];
```

### Reusable Types (Interfaces)

*Documentation*

> When specifying a greeting, you must pass a `GreetingSettings` object.
> This object has the following properties:
>
> 1 - greeting: Mandatory string
>
> 2 - duration: Optional length of time (in milliseconds)
>
> 3 - color: Optional string, e.g. '#ff00ff'

*Code*

```ts
greet({
  greeting: "hello world",
  duration: 4000
});
```

*Declaration*

Use an `interface` to define a type with properties.

```ts
interface GreetingSettings {
  greeting: string;
  duration?: number;
  color?: string;
}

declare function greet(setting: GreetingSettings): void;
```

### Reusable Types (Type Aliases)

*Documentation*

> Anywhere a greeting is expected, you can provide a `string`, a function returning a `string`, or a `Greeter` instance.

*Code*

```ts
function getGreeting() {
    return "howdy";
}
class MyGreeter extends Greeter { }

greet("hello");
greet(getGreeting);
greet(new MyGreeter());
```

*Declaration*

You can use a type alias to make a shorthand for a type:

```ts
type GreetingLike = string | (() => string) | MyGreeter;

declare function greet(g: GreetingLike): void;
```

### Organizing Types

*Documentation*

> The `greeter` object can log to a file or display an alert.
> You can provide LogOptions to `.log(...)` and alert options to `.alert(...)`

*Code*

```ts
const g = new Greeter("Hello");
g.log({ verbose: true });
g.alert({ modal: false, title: "Current Greeting" });
```

*Declaration*

Use namespaces to organize types.

```ts
declare namespace GreetingLib {
    interface LogOptions {
        verbose?: boolean;
    }
    interface AlertOptions {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

You can also create nested namespaces in one declaration:

```ts
declare namespace GreetingLib.Options {
    // Refer to via GreetingLib.Options.Log
    interface Log {
        verbose?: boolean;
    }
    interface Alert {
        modal: boolean;
        title?: string;
        color?: string;
    }
}
```

### Classes

*Documentation*

> You can create a greeter by instantiating the `Greeter` object, or create a customized greeter by extending from it.

*Code*

```ts
const myGreeter = new Greeter("hello, world");
myGreeter.greeting = "howdy";
myGreeter.showGreeting();

class SpecialGreeter extends Greeter {
    constructor() {
        super("Very special greetings");
    }
}
```

*Declaration*

Use `declare class` to describe a class or class-like object.
Classes can have properties and methods as well as a constructor.

```ts
declare class Greeter {
    constructor(greeting: string);

    greeting: string;
    showGreeting(): void;
}
```

<!-- Template

##

*Documentation*
>

*Code*

```ts

```

*Declaration*

```ts

```

-->

# Do's and Dont's

## General Types

### `Number`, `String`, `Boolean`, and `Object`

*Don't* ever use the types `Number`, `String`, `Boolean`, or `Object`.
These types refer to non-primitive boxed objects that are almost never used appropriately in JavaScript code.

```ts
/* WRONG */
function reverse(s: String): String;
```

*Do* use the types `number`, `string`, and `boolean`.

```ts
/* OK */
function reverse(s: string): string;
```

Instead of `Object`, use the non-primitive `object` type ([added in TypeScript 2.2](../release notes/TypeScript 2.2.md#object-type)).

### Generics

*Don't* ever have a generic type which doesn't use its type parameter.
See more details in [TypeScript FAQ page](https://github.com/Microsoft/TypeScript/wiki/FAQ#why-doesnt-type-inference-work-on-this-interface-interface-foot---).

<!-- TODO: More -->

## Callback Types

### Return Types of Callbacks

<!-- TODO: Reword; these examples make no sense in the context of a declaration file -->

*Don't* use the return type `any` for callbacks whose value will be ignored:

```ts
/* WRONG */
function fn(x: () => any) {
    x();
}
```

*Do* use the return type `void` for callbacks whose value will be ignored:

```ts
/* OK */
function fn(x: () => void) {
    x();
}
```

*Why*: Using `void` is safer because it prevents you from accidently using the return value of `x` in an unchecked way:

```ts
function fn(x: () => void) {
    var k = x(); // oops! meant to do something else
    k.doSomething(); // error, but would be OK if the return type had been 'any'
}
```

### Optional Parameters in Callbacks

*Don't* use optional parameters in callbacks unless you really mean it:

```ts
/* WRONG */
interface Fetcher {
    getObject(done: (data: any, elapsedTime?: number) => void): void;
}
```

This has a very specific meaning: the `done` callback might be invoked with 1 argument or might be invoked with 2 arguments.
The author probably intended to say that the callback might not care about the `elapsedTime` parameter,
  but there's no need to make the parameter optional to accomplish this --
  it's always legal to provide a callback that accepts fewer arguments.

*Do* write callback parameters as non-optional:

```ts
/* OK */
interface Fetcher {
    getObject(done: (data: any, elapsedTime: number) => void): void;
}
```

### Overloads and Callbacks

*Don't* write separate overloads that differ only on callback arity:

```ts
/* WRONG */
declare function beforeAll(action: () => void, timeout?: number): void;
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

*Do* write a single overload using the maximum arity:

```ts
/* OK */
declare function beforeAll(action: (done: DoneFn) => void, timeout?: number): void;
```

*Why*: It's always legal for a callback to disregard a parameter, so there's no need for the shorter overload.
Providing a shorter callback first allows incorrectly-typed functions to be passed in because they match the first overload.

## Function Overloads

### Ordering

*Don't* put more general overloads before more specific overloads:

```ts
/* WRONG */
declare function fn(x: any): any;
declare function fn(x: HTMLElement): number;
declare function fn(x: HTMLDivElement): string;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: any, wat?
```

*Do* sort overloads by putting the more general signatures after more specific signatures:

```ts
/* OK */
declare function fn(x: HTMLDivElement): string;
declare function fn(x: HTMLElement): number;
declare function fn(x: any): any;

var myElem: HTMLDivElement;
var x = fn(myElem); // x: string, :)
```

*Why*: TypeScript chooses the *first matching overload* when resolving function calls.
When an earlier overload is "more general" than a later one, the later one is effectively hidden and cannot be called.

### Use Optional Parameters

*Don't* write several overloads that differ only in trailing parameters:

```ts
/* WRONG */
interface Example {
    diff(one: string): number;
    diff(one: string, two: string): number;
    diff(one: string, two: string, three: boolean): number;
}
```

*Do* use optional parameters whenever possible:

```ts
/* OK */
interface Example {
    diff(one: string, two?: string, three?: boolean): number;
}
```

Note that this collapsing should only occur when all overloads have the same return type.

*Why*: This is important for two reasons.

TypeScript resolves signature compatibility by seeing if any signature of the target can be invoked with the arguments of the source,
  *and extraneous arguments are allowed*.
This code, for example, exposes a bug only when the signature is correctly written using optional parameters:

```ts
function fn(x: (a: string, b: number, c: number) => void) { }
var x: Example;
// When written with overloads, OK -- used first overload
// When written with optionals, correctly an error
fn(x.diff);
```

The second reason is when a consumer uses the "strict null checking" feature of TypeScript.
Because unspecified parameters appear as `undefined` in JavaScript, it's usually fine to pass an explicit `undefined` to a function with optional arguments.
This code, for example, should be OK under strict nulls:

```ts
var x: Example;
// When written with overloads, incorrectly an error because of passing 'undefined' to 'string'
// When written with optionals, correctly OK
x.diff("something", true ? undefined : "hour");
```

### Use Union Types

*Don't* write overloads that differ by type in only one argument position:

```ts
/* WRONG */
interface Moment {
    utcOffset(): number;
    utcOffset(b: number): Moment;
    utcOffset(b: string): Moment;
}
```

*Do* use union types whenever possible:

```ts
/* OK */
interface Moment {
    utcOffset(): number;
    utcOffset(b: number|string): Moment;
}
```

Note that we didn't make `b` optional here because the return types of the signatures differ.

*Why*: This is important for people who are "passing through" a value to your function:

```ts
function fn(x: string): void;
function fn(x: number): void;
function fn(x: number|string) {
    // When written with separate overloads, incorrectly an error
    // When written with union types, correctly OK
    return moment().utcOffset(x);
}
```

# Deep Dive

## Definition File Theory: A Deep Dive

Structuring modules to give the exact API shape you want can be tricky.
For example, we might want a module that can be invoked with or without `new` to produce different types,
  has a variety of named types exposed in a hierarchy,
  and has some properties on the module object as well.

By reading this guide, you'll have the tools to write complex definition files that expose a friendly API surface.
This guide focuses on module (or UMD) libraries because the options here are more varied.

### Key Concepts

You can fully understand how to make any shape of definition
  by understanding some key concepts of how TypeScript works.

#### Types

If you're reading this guide, you probably already roughly know what a type in TypeScript is.
To be more explicit, though, a *type* is introduced with:

* A type alias declaration (`type sn = number | string;`)
* An interface declaration (`interface I { x: number[]; }`)
* A class declaration (`class C { }`)
* An enum declaration (`enum E { A, B, C }`)
* An `import` declaration which refers to a type

Each of these declaration forms creates a new type name.

#### Values

As with types, you probably already understand what a value is.
Values are runtime names that we can reference in expressions.
For example `let x = 5;` creates a value called `x`.

Again, being explicit, the following things create values:

* `let`, `const`, and `var` declarations
* A `namespace` or `module` declaration which contains a value
* An `enum` declaration
* A `class` declaration
* An `import` declaration which refers to a value
* A `function` declaration

#### Namespaces

Types can exist in *namespaces*.
For example, if we have the declaration `let x: A.B.C`,
  we say that the type `C` comes from the `A.B` namespace.

This distinction is subtle and important -- here, `A.B` is not necessarily a type or a value.

### Simple Combinations: One name, multiple meanings

Given a name `A`, we might find up to three different meanings for `A`: a type, a value or a namespace.
How the name is interpreted depends on the context in which it is used.
For example, in the declaration `let m: A.A = A;`,
  `A` is used first as a namespace, then as a type name, then as a value.
These meanings might end up referring to entirely different declarations!

This may seem confusing, but it's actually very convenient as long as we don't excessively overload things.
Let's look at some useful aspects of this combining behavior.

#### Built-in Combinations

Astute readers will notice that, for example, `class` appeared in both the *type* and *value* lists.
The declaration `class C { }` creates two things:
  a *type* `C` which refers to the instance shape of the class,
  and a *value* `C` which refers to the constructor function of the class.
Enum declarations behave similarly.

#### User Combinations

Let's say we wrote a module file `foo.d.ts`:

```ts
export var SomeVar: { a: SomeType };
export interface SomeType {
  count: number;
}
```

Then consumed it:

```ts
import * as foo from './foo';
let x: foo.SomeType = foo.SomeVar.a;
console.log(x.count);
```

This works well enough, but we might imagine that `SomeType` and `SomeVar` were very closely related
  such that you'd like them to have the same name.
We can use combining to present these two different objects (the value and the type) under the same name `Bar`:

```ts
export var Bar: { a: Bar };
export interface Bar {
  count: number;
}
```

This presents a very good opportunity for destructuring in the consuming code:

```ts
import { Bar } from './foo';
let x: Bar = Bar.a;
console.log(x.count);
```

Again, we've used `Bar` as both a type and a value here.
Note that we didn't have to declare the `Bar` value as being of the `Bar` type -- they're independent.

### Advanced Combinations

Some kinds of declarations can be combined across multiple declarations.
For example, `class C { }` and `interface C { }` can co-exist and both contribute properties to the `C` types.

This is legal as long as it does not create a conflict.
A general rule of thumb is that values always conflict with other values of the same name unless they are declared as `namespace`s,
  types will conflict if they are declared with a type alias declaration (`type s = string`),
  and namespaces never conflict.

Let's see how this can be used.

#### Adding using an `interface`

We can add additional members to an `interface` with another `interface` declaration:

```ts
interface Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

This also works with classes:

```ts
class Foo {
  x: number;
}
// ... elsewhere ...
interface Foo {
  y: number;
}
let a: Foo = ...;
console.log(a.x + a.y); // OK
```

Note that we cannot add to type aliases (`type s = string;`) using an interface.

#### Adding using a `namespace`

A `namespace` declaration can be used to add new types, values, and namespaces in any way which does not create a conflict.

For example, we can add a static member to a class:

```ts
class C {
}
// ... elsewhere ...
namespace C {
  export let x: number;
}
let y = C.x; // OK
```

Note that in this example, we added a value to the *static* side of `C` (its constructor function).
This is because we added a *value*, and the container for all values is another value
  (types are contained by namespaces, and namespaces are contained by other namespaces).

We could also add a namespaced type to a class:

```ts
class C {
}
// ... elsewhere ...
namespace C {
  export interface D { }
}
let y: C.D; // OK
```

In this example, there wasn't a namespace `C` until we wrote the `namespace` declaration for it.
The meaning `C` as a namespace doesn't conflict with the value or type meanings of `C` created by the class.

Finally, we could perform many different merges using `namespace` declarations.
This isn't a particularly realistic example, but shows all sorts of interesting behavior:

```ts
namespace X {
  export interface Y { }
  export class Z { }
}

// ... elsewhere ...
namespace X {
  export var Y: number;
  export namespace Z {
    export class C { }
  }
}
type X = string;
```

In this example, the first block creates the following name meanings:

* A value `X` (because the `namespace` declaration contains a value, `Z`)
* A namespace `X` (because the `namespace` declaration contains a type, `Y`)
* A type `Y` in the `X` namespace
* A type `Z` in the `X` namespace (the instance shape of the class)
* A value `Z` that is a property of the `X` value (the constructor function of the class)

The second  block creates the following name meanings:

* A value `Y` (of type `number`) that is a property of the `X` value
* A namespace `Z`
* A value `Z` that is a property of the `X` value
* A type `C` in the `X.Z` namespace
* A value `C` that is a property of the `X.Z` value
* A type `X`

### Using with `export =` or `import`

An important rule is that `export` and `import` declarations export or import *all meanings* of their targets.

<!-- TODO: Write more on that. -->

# Publishing

Now that you have authored a declaration file following the steps of this guide, it is time to publish it to npm.
There are two main ways you can publish your declaration files to npm:

1. bundling with your npm package, or
2. publishing to the [@types organization](https://www.npmjs.com/~types) on npm.

If your package is written in TypeScript then the first approach is favored.
Use the `--declaration` flag to generate declaration files.
This way, your declarations and JavaScript always be in sync.

If your package is not written in TypeScript then the second is the preferred approach.

## Including declarations in your npm package

If your package has a main `.js` file, you will need to indicate the main declaration file in your `package.json` file as well.
Set the `types` property to point to your bundled declaration file.
For example:

```json
{
    "name": "awesome",
    "author": "Vandelay Industries",
    "version": "1.0.0",
    "main": "./lib/main.js",
    "types": "./lib/main.d.ts"
}
```

Note that the `"typings"` field is synonymous with `"types"`, and could be used as well.

Also note that if your main declaration file is named `index.d.ts` and lives at the root of the package (next to `index.js`) you do not need to mark the `"types"` property, though it is advisable to do so.

### Dependencies

All dependencies are managed by npm.
Make sure all the declaration packages you depend on are marked appropriately in the `"dependencies"` section in your `package.json`.
For example, imagine we authored a package that used Browserify and TypeScript.

```json
{
    "name": "browserify-typescript-extension",
    "author": "Vandelay Industries",
    "version": "1.0.0",
    "main": "./lib/main.js",
    "types": "./lib/main.d.ts",
    "dependencies": {
        "browserify": "latest",
        "@types/browserify": "latest",
        "typescript": "next"
    }
}
```

Here, our package depends on the `browserify` and `typescript` packages.
`browserify` does not bundle its declaration files with its npm packages, so we needed to depend on `@types/browserify` for its declarations.
`typescript`, on the other hand, packages its declaration files, so there was no need for any additional dependencies.

Our package exposes declarations from each of those, so any user of our `browserify-typescript-extension` package needs to have these dependencies as well.
For that reason, we used `"dependencies"` and not `"devDependencies"`, because otherwise our consumers would have needed to manually install those packages.
If we had just written a command line application and not expected our package to be used as a library, we might have used `devDependencies`.

### Red flags

#### `/// <reference path="..." />`

*Don't* use `/// <reference path="..." />` in your declaration files.

```ts
/// <reference path="../typescript/lib/typescriptServices.d.ts" />
....
```

*Do* use `/// <reference types="..." />` instead.

```ts
/// <reference types="typescript" />
....
```

Make sure to revisit the [Consuming dependencies](./Library Structures.md#consuming-dependencies) section for more information.

#### Packaging dependent declarations

If your type definitions depend on another package:

* *Don't* combine it with yours, keep each in their own file.
* *Don't* copy the declarations in your package either.
* *Do* depend on the npm type declaration package if it doesn't package its declaration files.

## Publish to [@types](https://www.npmjs.com/~types)

Packages on under the [@types](https://www.npmjs.com/~types) organization are published automatically from [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) using the [types-publisher tool](https://github.com/Microsoft/types-publisher).
To get your declarations published as an @types package, please submit a pull request to [https://github.com/DefinitelyTyped/DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped).
You can find more details in the [contribution guidelines page](http://definitelytyped.org/guides/contributing.html).

# Consumption

In TypeScript 2.0, it has become significantly easier to consume declaration files, in acquiring, using, and finding them.
This page details exactly how to do all three.

## Downloading

Getting type declarations in TypeScript 2.0 and above requires no tools apart from npm.

As an example, getting the declarations for a library like lodash takes nothing more than the following command

```cmd
npm install --save @types/lodash
```

It is worth noting that if the npm package already includes its declaration file as described in [Publishing](./Publishing.md), downloading the corresponding `@types` package is not needed.

## Consuming

From there you’ll be able to use lodash in your TypeScript code with no fuss.
This works for both modules and global code.

For example, once you’ve `npm install`-ed your type declarations, you can use imports and write

```ts
import * as _ from "lodash";
_.padStart("Hello TypeScript!", 20, " ");
```

or if you’re not using modules, you can just use the global variable `_`.

```ts
_.padStart("Hello TypeScript!", 20, " ");
```

## Searching

For the most part, type declaration packages should always have the same name as the package name on `npm`, but prefixed with `@types/`,
  but if you need, you can check out [https://aka.ms/types](https://aka.ms/types) to find the package for your favorite library.

> Note: if the declaration file you are searching for is not present, you can always contribute one back and help out the next developer looking for it.
> Please see the DefinitelyTyped [contribution guidelines page](http://definitelytyped.org/guides/contributing.html) for details.

# Templates

## global-modifying-module.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the global-modifying module template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ Note: If your global-modifying module is callable or constructable, you'll
 *~ need to combine the patterns here with those in the module-class or module-function
 *~ template files
 */
declare global {
    /*~ Here, declare things that go in the global namespace, or augment
     *~ existing declarations in the global namespace
     */
    interface String {
        fancyFormat(opts: StringFormatOptions): string;
    }
}

/*~ If your module exports types or values, write them as usual */
export interface StringFormatOptions {
    fancinessLevel: number;
}

/*~ For example, declaring a method on the module (in addition to its global side effects) */
export function doSomething(): void;

/*~ If your module exports nothing, you'll need this line. Otherwise, delete it */
export { };
```

## global-plugin.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This template shows how to write a global plugin. */

/*~ Write a declaration for the original type and add new members.
 *~ For example, this adds a 'toBinaryString' method with to overloads to
 *~ the built-in number type.
 */
interface Number {
    toBinaryString(opts?: MyLibrary.BinaryFormatOptions): string;
    toBinaryString(callback: MyLibrary.BinaryFormatCallback, opts?: MyLibrary.BinaryFormatOptions): string;
}

/*~ If you need to declare several types, place them inside a namespace
 *~ to avoid adding too many things to the global namespace.
 */
declare namespace MyLibrary {
    type BinaryFormatCallback = (n: number) => string;
    interface BinaryFormatOptions {
        prefix?: string;
        padding: number;
    }
}
```

## global.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ If this library is callable (e.g. can be invoked as myLib(3)),
 *~ include those call signatures here.
 *~ Otherwise, delete this section.
 */
declare function myLib(a: string): string;
declare function myLib(a: number): number;

/*~ If you want the name of this library to be a valid type name,
 *~ you can do so here.
 *~
 *~ For example, this allows us to write 'var x: myLib';
 *~ Be sure this actually makes sense! If it doesn't, just
 *~ delete this declaration and add types inside the namespace below.
 */
interface myLib {
    name: string;
    length: number;
    extras?: string[];
}

/*~ If your library has properties exposed on a global variable,
 *~ place them here.
 *~ You should also place types (interfaces and type alias) here.
 */
declare namespace myLib {
    //~ We can write 'myLib.timeout = 50;'
    let timeout: number;

    //~ We can access 'myLib.version', but not change it
    const version: string;

    //~ There's some class we can create via 'let c = new myLib.Cat(42)'
    //~ Or reference e.g. 'function f(c: myLib.Cat) { ... }
    class Cat {
        constructor(n: number);

        //~ We can read 'c.age' from a 'Cat' instance
        readonly age: number;

        //~ We can invoke 'c.purr()' from a 'Cat' instance
        purr(): void;
    }

    //~ We can declare a variable as
    //~   'var s: myLib.CatSettings = { weight: 5, name: "Maru" };'
    interface CatSettings {
        weight: number;
        name: string;
        tailLength?: number;
    }

    //~ We can write 'const v: myLib.VetID = 42;'
    //~  or 'const v: myLib.VetID = "bob";'
    type VetID = string | number;

    //~ We can invoke 'myLib.checkCat(c)' or 'myLib.checkCat(c, v);'
    function checkCat(c: Cat, s?: VetID);
}
```

## module-class.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file for class modules.
 *~ You should rename it to index.d.ts and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ Note that ES6 modules cannot directly export class objects.
 *~ This file should be imported using the CommonJS-style:
 *~   import x = require('someLibrary');
 *~
 *~ Refer to the documentation to understand common
 *~ workarounds for this limitation of ES6 modules.
 */

/*~ If this module is a UMD module that exposes a global variable 'myClassLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myClassLib;

/*~ This declaration specifies that the class constructor function
 *~ is the exported object from the file
 */
export = MyClass;

/*~ Write your module's methods and properties in this class */
declare class MyClass {
    constructor(someParam?: string);

    someProperty: string[];

    myMethod(opts: MyClass.MyClassMethodOptions): number;
}

/*~ If you want to expose types from your module as well, you can
 *~ place them in this block.
 */
declare namespace MyClass {
    export interface MyClassMethodOptions {
        width?: number;
        height?: number;
    }
}
```

## module-function.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file for function modules.
 *~ You should rename it to index.d.ts and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ Note that ES6 modules cannot directly export callable functions.
 *~ This file should be imported using the CommonJS-style:
 *~   import x = require('someLibrary');
 *~
 *~ Refer to the documentation to understand common
 *~ workarounds for this limitation of ES6 modules.
 */

/*~ If this module is a UMD module that exposes a global variable 'myFuncLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myFuncLib;

/*~ This declaration specifies that the function
 *~ is the exported object from the file
 */
export = MyFunction;

/*~ This example shows how to have multiple overloads for your function */
declare function MyFunction(name: string): MyFunction.NamedReturnType;
declare function MyFunction(length: number): MyFunction.LengthReturnType;

/*~ If you want to expose types from your module as well, you can
 *~ place them in this block. Often you will want to describe the
 *~ shape of the return type of the function; that type should
 *~ be declared in here, as this example shows.
 */
declare namespace MyFunction {
    export interface LengthReturnType {
        width: number;
        height: number;
    }
    export interface NamedReturnType {
        firstName: string;
        lastName: string;
    }

    /*~ If the module also has properties, declare them here. For example,
     *~ this declaration says that this code is legal:
     *~   import f = require('myFuncLibrary');
     *~   console.log(f.defaultName);
     */
    export const defaultName: string;
    export let defaultLength: number;
}
```

## module-plugin.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module plugin template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ On this line, import the module which this module adds to */
import * as m from 'someModule';

/*~ You can also import other modules if needed */
import * as other from 'anotherModule';

/*~ Here, declare the same module as the one you imported above */
declare module 'someModule' {
    /*~ Inside, add new function, classes, or variables. You can use
     *~ unexported types from the original module if needed. */
    export function theNewMethod(x: m.foo): other.bar;

    /*~ You can also add new properties to existing interfaces from
     *~ the original module by writing interface augmentations */
    export interface SomeModuleOptions {
        someModuleSetting?: string;
    }

    /*~ New types can also be declared and will appear as if they
     *~ are in the original module */
    export interface MyModulePluginOptions {
        size: number;
    }
}
```

## module.d.ts

```ts
// Type definitions for [~THE LIBRARY NAME~] [~OPTIONAL VERSION NUMBER~]
// Project: [~THE PROJECT NAME~]
// Definitions by: [~YOUR NAME~] <[~A URL FOR YOU~]>

/*~ This is the module template file. You should rename it to index.d.ts
 *~ and place it in a folder with the same name as the module.
 *~ For example, if you were writing a file for "super-greeter", this
 *~ file should be 'super-greeter/index.d.ts'
 */

/*~ If this module is a UMD module that exposes a global variable 'myLib' when
 *~ loaded outside a module loader environment, declare that global here.
 *~ Otherwise, delete this declaration.
 */
export as namespace myLib;

/*~ If this module has methods, declare them as functions like so.
 */
export function myMethod(a: string): string;
export function myOtherMethod(a: number): number;

/*~ You can declare types that are available via importing the module */
export interface someType {
    name: string;
    length: number;
    extras?: string[];
}

/*~ You can declare properties of the module using const, let, or var */
export const myField: number;

/*~ If there are types, properties, or methods inside dotted names
 *~ of the module, declare them inside a 'namespace'.
 */
export namespace subProp {
    /*~ For example, given this definition, someone could write:
     *~   import { subProp } from 'yourModule';
     *~   subProp.foo();
     *~ or
     *~   import * as yourMod from 'yourModule';
     *~   yourMod.subProp.foo();
     */
    export function foo(): void;
}
```
