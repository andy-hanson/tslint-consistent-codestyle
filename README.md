# Purpose

The rules in this package can be used to enforce consistent code style.

# Usage

Install from npm to your devDependencies  (https://www.npmjs.com/package/tslint-consistent-codestyle)

```
npm install --save-dev tslint-consistent-codestyle
```

Configure tslint to use the tslint-consistent-codestyle folder:

Add the following path to the `rulesDirectory` setting in your `tslint.json` file:

```javascript
{
   "rulesDirectory": ["node_modules/tslint-consistent-codestyle/rules"]
   "rules": {
     ...
   }
}
```

Now configure some of the new rules.

# Rules

## naming-convention
Enforce consistent names for almost everything.

This rule is configured with an array of configuration objects. All of those objects are made up of 2 parts. The first could be called the "selector", because it describes, *what* is affected by this config. The second part consists of one or more formatting rules.

So for example, if you want to force every local variable to be in camelCase, you simply write the config as shown below:
```javascript
{
  "type": "variable",
  "modifiers": "local",
  "format": "camelCase"
}
```

__Selector__:

* `type: string`: The selector consists of a required field `type`, which identifies the type of thing this configuration applies to.
* `modifiers?: string|string[]`: To further specify your selection you can provide one or more `modifiers`. All of those modifiers must match an identifier, to activate this config. That means `["global", "const"]` will match every constant in global scope. It will not match any not-const global. Some of those modifiers are mutually exclusive, so you will never match anything if you specify more than one of them, for example `global` and `local` or the access modifiers `private`, `protected` and `public`.
* `final?: boolean`: If set to true, this configuration will not contribute to the composition of any subtype's configuration.

__Formatting rules__:

All formatting rules are optional. Formatting rules are inherited by the `type`'s parent type. You can shadow the parent config by setting a new rule for the desired check or even disable the check by setting any falsy value other than `undefined`.

* `regex: string`: A regular expression, which is applied to the name. (without any regex flags)
* `leadingUnderscore: string, trailingUnderscore: string`: Options `forbid`, `allow` and `require` can be used to forbid, allow or require _one_ leading or trailing underscore in the name. **If one option is specified, the leading or trailing underscore will be sliced off the name before any further checks are performed.**
* `prefix: string|string[], suffix: string|string[]`: Specify one or more prefixes or suffixes. When given a single string, that string must match in the specified position. When given an array, one of the strings must match in the specified position. Matching is done in the given order. **If a prefix or suffix is specified, the matching portion of the name is sliced off before any further checks are performed:** If you enforce `cascalCase` and a prefix `has`, the name `hasFoo` will not match. That's because the prefix `has` is removed and the remaining `Foo` is not valid `camelCase`  
* `format: string`: Valid options are `camelCase`, `PascalCase`, `UPPER_CASE` and `snake_case`.

__"Inheritance" / Extending configurations:__

As mentioned above, a type's configuration is used as a base for the configuration of all of it's subtypes. Of course the subtype can override any inherited configuration option by providing a new value or disable it by setting a falsy value other than `undefined`.

For example there is a base type `default` that applies to every other type, if it is not declared as `final`.
We will cover that concept later in the examples section.

### Types
___default___

* Scope: is the base for everything
* Valid modifiers: refer to subtypes


___variable___

* Scope: every variable declared with `var`, `let` and `const`
* Extends: `default`
* Valid modifiers:
  * `global` or `local`
  * `const`
  * `export`
  * `rename` // if you rename a property in a destructuring assignment, e.g. `let {foo: myFoo} = bar`


___function___

* Scope: every function and named function expression
* Extends: `variable`
* Valid modifiers:
  * `global` or `local`
  * `export`


___parameter___

* Scope: parameters
* Extends: `variable`, always has `local` modifier
* Valid modifiers:
  * `rename`


___member___

* Scope: used as superclass for `property`, `method`, ...
* Extends: `default`, always has `local` modifier
* Valid modifiers: refer to subtypes


___property___

* Scope: class properties (static and instance)
* Extends: `member`
* Valid modifiers:
  * `private`, `protected` or `public`
  * `static`
  * `const` == `readonly` // can be used interchangably. internally both are handled as `const`
  * `abstract` // for abstract property accessors


___parameterProperty___

* Scope: class constructor's parameter properties
* Extends: `property` and `parameter`
* Valid modifiers:
  * `private`, `protected` or `public`
  * `const` == `readonly`


___enumMember___

* Scope: all members of enums
* Extends: `property`, always has `public`, `static`, `const`/`readonly` modifiers
* Valid modifiers:
  * everything from type `enum`


___method___

* Scope: class methods (static and instance)
* Extends: `member`
* Valid modifiers:
  * `private`, `protected` or `public`
  * `static`
  * `abstract`


___type___

* Scope: used as superclass for `class`, `interface`, ...
* Extends: `default`
* Valid modifiers: refer to subtypes


___class___

* Scope: all classes and named class expressions
* Extends: `type`
* Valid modifiers:
  * `global` or `local`
  * `abstract`
  * `export`


___interface___

* Scope: all interfaces
* Extends: `type`
* Valid modifiers:
  * `global` or `local`
  * `export`


___typeAlias___

* Scope: all type aliases, e.g. `type Foo = "a" | "b"`
* Extends: `type`
* Valid modifiers:
  * `global` or `local`
  * `export`


___genericTypeParameter___

* Scope: all generic type parameters, e.g. `class Foo<T, U> {}` or `function<T>(v: T): T {}`
* Extends: `type`
* Valid modifiers:
  * `global` _if found on a class_ or `local` _if found on a function_


___enum___

* Scope: all enums
* Extends: `type`
* Valid modifiers:
  * `global` or `local`
  * `const`
  * `export`

## no-collapsible-if
*Tired of unnecessarily deep nested if blocks?*
This rule comes to rescue you.

Not passing:
```javascript
/* 1 */
if (foo)
  if (bar);
/* 2 */
if (foo) {
  if (bar) {
  }
}
/* 3 */
if (foo) {
} else {
  if (bar) {
  } else {
  }
}
```
Passing:
```javascript
/* 1 */
if (foo && bar);
/* 2 */
if (foo && bar) {
}
/* 3 */
if (foo) {
} else if (bar) {
} else {
}

if (foo) {
  if (bar) {
  } else {
  }
}
```
I recommend you use this rule together with [no-else-after-return](#no-else-after-return) or [no-unnecessary-else](#no-unnecessary-else) to further reduce block nesting.

## no-else-after-return
Works like [no-else-return from eslint](http://eslint.org/docs/rules/no-else-return).

> If an if block contains a return statement, the else block becomes unnecessary. Its contents can be placed outside of the block.

If you like this rule, I recommend you try [no-unnecessary-else](#no-unnecessary-else) for some bonus features.

## no-return-undefined
Using `return undefined` or `return void 0` is unnecessary, because `undefined` is the default return value. Just use `return;` instead.

## no-static-this
Ban the use of `this` in static methods.

__Rationale:__ It's pretty hard to wrap your head around the meaning of `this` in a static context. Especially newcomers will not recognise, that you are in fact referencing the `class` (or the `constructor` to be more precise).

## no-unnecessary-else
Works like [no-else-after-return](#no-else-after-return) with additional checks. This rule gives you hints for unnecessary else blocks after `throw`, `continue` and `break`(if not used within a switch block) for free. Just change `no-else-after-return` to `no-unnecessary-else` in your `tslint.json`.

## no-var-before-return
> Declaring a variable only to immediately return it is a **bad practice**. Some developers argue that the practice improves code readability, because it enables them to explicitly name what is being returned. However, this variable is an internal implementation detail that is not exposed to the callers of the method. **The method name should be sufficient for callers to know exactly what will be returned.**

This rule checks, if the last variable declared in a variable declaration right before the return statement contains the returned variable.
Destructuring assignments are also checked because `let {foo} = bar; return foo;` can also be written as `return bar.foo;`.
But if the destructuring assignment for this variable contains a default value other than `undefined` or `void`, there will be no error.

## parameter-properties
Usage:
```javascript
"parameter-properties": [
  true,
  "all-or-none",   // forces all or none of a constructors parameters to be parameter properties
  "leading",       // forces parameter properties to precede regular parameters
  "member-access", // forces an access modifier for every parameter property
  "readonly"       // forces all parameter properties to be readonly
]
```

All rule options are optional, but without any option this rule does nothing.

## prefer-const-enum [WIP]
Flags all enum declarations, if enum can be declared as constant. That is: there is no dynamic member access and the enum object is never assigned or passed to anything.

__Current limitations:__
* no support for scopes
* doesn't ignore exported enums (maybe add option or use `preserveConstEnums` compiler flag)
* doesn't detect if enum is assigned to variable or passed as function parameter

## prefer-while
A `for`-loop without initializer and incrementer can also be rewritten as `while`-loop. That's what this rule does.
It can also automatically fix any finding, if you set the `--fix` command line switch for `tslint`.