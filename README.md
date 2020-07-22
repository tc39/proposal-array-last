# Getting last item from Array

[(Currently Stage 1)](https://github.com/tc39/proposals/blob/master/stage-1-proposals.md)

This proposal's champion (@keithamus) does not plan to advance to Stage 2 for now. Other proposals ([Array.prototype.item](https://github.com/tabatkins/proposal-item-method) and [Array Slice Notation](https://github.com/tc39/proposal-slice-notation)) also sufficiently solve this problem, and are advancing quicker through the standards track. Should one of these proposals advance to Stage 3, this proposal will be dropped.

## Rationale

Currently the only way to get the last element of an Array is to subtract one from the `.length` property and use this value in the property lookup. Typically this looks like `someArray[someArray.length - 1]`.

For such a common operation, it is easy to forget to `-1` from the `length` property, or to overcook it, and as such there should be a simpler syntax for doing this.

```js
myArray[myArray.length] // oops, index out of bounds, return `undefined`, scratch head for hours from silly mistake


myIndex = myArray.length - 1
myArray[myIndex - 1] // oops, overcooked index, returns last-but-one not last, scratch head for hours from silly mistake
```

## High Level Proposal

Array.prototype.lastItem is a property with a getter/setter function that returns the last item of the Array.
Array.prototype.lastIndex is a property with a getter function that returns the last index of the Array.

## Specification

```
22.1.3.xx Array.prototype.lastItem

It is a property where the attributes are { [[Enumerable]]: false, [[Configurable]]: false, [[Get]]: GetLastArrayItem, [[Set]]: SetLastArrayItem }.

22.1.3.xx GetLastArrayItem 

When the GetLastArrayItem method is called, the following steps are taken:

    Let O be ? ToObject(this value).
    Let len be ? ToLength(? Get(O, "length")).
    If len is zero, then
        Return undefined.
    Else len > 0,
        Set len to len-1.
        Let index be ! ToString(len).
        Let element be ? Get(O, index).
        Return element. 

The GetLastArrayItem function is intentionally generic; it does not require that its this value be an Array object. Therefore it can be transferred to other kinds of objects for use as a method.

22.1.3.xx SetLastArrayItem (value)

When the SetLastArrayItem method is called, the following steps are taken:

    Let O be ? ToObject(this value).
    Let len be ? ToLength(? Get(O, "length")).
    If len > 0, then
        Set len to len-1.
    Let index be ! ToString(len).
    Return ? Set(O, index, value).

Note 1

The SetLastArrayItem function is intentionally generic; it does not require that its this value be an Array object. Therefore it can be transferred to other kinds of objects for use as a method.

22.1.3.xx Array.prototype.lastIndex

It is a property where the attributes are { [[Enumerable]]: false, [[Configurable]]: false, [[Get]]: GetLastArrayIndex }.

22.1.3.xx GetLastArrayIndex 

When the GetLastArrayIndex method is called, the following steps are taken:

    Let O be ? ToObject(this value).
    Let len be ? ToLength(? Get(O, "length")).
    If len > 0, then
        Return len-1.
    Return 0.

Note 1

The GetLastArrayIndex function is intentionally generic; it does not require that its this value be an Array object. Therefore it can be transferred to other kinds of objects for use as a method.
```

## Polyfill

A polyfill is available in the [core-js](https://github.com/zloirock/core-js) library. You can find it in the [ECMAScript proposals section](https://github.com/zloirock/core-js#getting-last-item-from-array).

It's a polyfill with usage abstract ECMAScript operations:
```js
import { ToString, ToObject, ToLength } from 'es-abstract'
// This polyfill tries to stick as close to the spec as possible. There are polyfills which could use less code.
Object.defineProperty(Array.prototype, 'lastItem', {
  enumerable: false,
  configurable: false,
  get() {
    let O = ToObject(this)
    let len = ToLength(O.length)
    if (len === 0) {
      return undefined
    } else if (len > 0) {
      len = len -1
      let index = ToString(len)
      let element = O[index]
      return element
    }
  },
  set(value) {
    let O = ToObject(this)
    let len = ToLength(O.length)
    if (len > 0) {
      len = len -1
    }
    let index = ToString(len)
    return O[index] = value
  },
})
Object.defineProperty(Array.prototype, 'lastIndex', {
  enumerable: false,
  configurable: false,
  get() {
    let O = ToObject(this)
    let len = ToLength(O.length)
    if (len > 0) {
      return len - 1
    }
    return 0
  },
})
```

#### Other considered options

 - `Array.prototype.last`/`Array.prototype.last()` was originally proposed but has web-compatibility issues.
 - `Array.prototype.end()` as a method. The downside being that setting the last element of an array is still uses awkward syntax.
 - `Array.prototype.end` as a getter. However `lastItem` was a more popular name choice.
 - `Array.prototype.peek()` was considered (marries well with `push`, `pop`) but `peek` as a setter is unintuitive.
 - `Array.prototype.prePop()` [was mentioned](https://github.com/keithamus/proposal-array-last/issues/11#issuecomment-372933559) but suffers from the same issues as `peek` - it is unintuitive for setting, and potentially surprising if it _doesn't_ mutate. 
 - [A bunch of other names](https://github.com/keithamus/proposal-array-last/issues/11#issuecomment-362246040). Most importantly see the following rational for a naming rubric:
   - Not have webcompat issues
   - Within the top 1000 most popular english words (both last and end are)
   - Should be intuitive for both getting and setting
   - Ideally not be a compound word (like lastItem)
