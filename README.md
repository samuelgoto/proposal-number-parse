# Number.parse()

# Stage 0

This is a [stage 0](https://tc39.github.io/process-document/) proposal to create a static method in Number that:

* parses a string (ignoring non-numeric suffix, viz parseFloat) and returns a Number and
* stays up to date with the evolution of numeric literals (viz the Number constructor)

# Motivation

As the language evolves and new syntax for numeric literals are added (e.g. [binary/octal numeric literals](https://esdiscuss.org/topic/a-new-es6-draft-rev28), [numeric literal separators](https://github.com/tc39/proposal-numeric-separator), etc) more and more parseFloat and parseInt become outdated.

The Number() constructor keeps up to date to the syntax to parse numbers, but can't be used to parse strings as effectively as parseFloat/parseInt can (specifically, it returns NaN on non-numeric suffixes).

```javascript
parseInt("0b11") === 0; // expected 3, doesn't keep up with the newly introduced binary prefix
Number("1px") === NaN; // expected 1, doesn't ignore the non-numeric "px" suffix
```

Ideally, we would like something that:

* resembles parseFloat/parseInt in that it can take input that isn't strictly numeric and parse it appropriately
* resembles Number() in that new syntax introduce propagates to the parser

Hence:

```javascript
Number.parse("0b11") == 3; // Yay, Number.parse keeps up with new syntax!
Number.parse("1px") == 1; // Yay, Number.parse is more useful than Number()!
```

# Examples

```js
// As opposed to parseFloat("1_000") === 1000, Number.parse ignores _
Number.parse("1_000") === 1000; 

// As opposed to Number("1px") === NaN, Number.parse takes valid prefix
Number.parse("1px") === 1; 

// As with parseFloat("Infinity") === Infinity, Number.parse special cases Infinity.
Number.parse("Infinity") === Infinity;

// As opposed to parseFloat("0xFF") === 0, Number.parse takes radixes prefixes
Number.parse("0xFF") === 255; 

// As opposed to Number.parseInt("011", 8) == 9, but similar to Number("011") === 11,
// Number.parse assumes numeric literals leading with 0s to be decimal
Number.parse("011") === 11;

// As with Number("0o11") == 3 and Number("0b11") == 3, Number.parse can
// also parse binary, octal and hexadecimal prefixes
Number.parse("0o11") == 9;

// As with Number(" 0xF") === 15, Number.parse ignores leading whitespace.
Number.parse(" 0xF") === 15;
```

# Background

This proposal is attempting to resolve a long standing [issue](https://github.com/tc39/ecma262/issues/927) brought up when binary/octal literals were introduced and parseInt didn't pick up the syntax changes.

More recently, as numeric separators are moving on, a similar [issue](https://github.com/tc39/proposal-numeric-separator/issues/11) was brought up that parseFloat() wasn't useful anymore to parse user input with _s.

In addition to that, the same [issue](https://github.com/tc39/proposal-numeric-separator/issues/11#issuecomment-314893789) appears as we introduce BigInts.

# Alternatives considered

Here are some of the ideas that circulated in different threads:

## Backwards incompatible APIs

  * Modifying the Number() constructor to stop at non-numeric suffixes    
  * Modifying the parseFloat() method to be up to date to parsing numeric literals (e.g. binary/octal literals and numeric separators)
    
```javascript
// Backwards-incompatible change to make Number() stop at non-numeric 
// suffixes.
Number("1px") === 1; // From Number("1px") === NaN

// Backwards-incompatible change to make parseFloat() keep up 
// to date with numeric literals.
parseFloat("0xF") === 15; // From parseFloat("0xF") === 0.
```

## Explicit APIs

  * Extending Number() with explicit parameters (e.g. [radix](https://github.com/tc39/ecma262/issues/927#issuecomment-306629879))
  * [explicit parameters](https://github.com/tc39/ecma262/issues/927#issuecomment-306595340)

Here are some examples:

```javascript

// Extending Number() with explicit parameters.
Number('0123', 4) === parseInt('0123', 4) === 27;

// Explicit parameters.
Number.parseInt('0b10', undefined, {
  binary:true, 
  octal:true, 
  detectRepeats:true
})

```

## Polyfills

# Specification 

TODO(goto): fill this in once we get into stage 1.

# References

* [Numeric Separator](https://github.com/tc39/proposal-numeric-separator/issues/11)
* [Binary literals](https://github.com/tc39/ecma262/issues/927)

