# Getting last item from Array

Currently the only way to get the last element of an Array is to subtract one from the `.length` property and use this value in the property lookup. Typically this looks like `someArray[someArray.length - 1]`.

For such a common operation, it is easy to forget to `-1` from the `length` property, or to overcook it, and as such there should be a simpler syntax for doing this.

```js
myArray[myArray.length] // oops, index out of bounds, return `undefined`, scratch head for hours from silly mistake


myIndex = myArray.length - 1
myArray[myIndex - 1] // oops, overcooked index, returns last-but-one not last, scratch head for hours from silly mistake
```

## Options

There exist a few options for getting the last element from an Array:

### Make `-1` a meta index to retrieve the last property

#### High Level Semantics

Array.prototype['-1'] is a property with a getter/setter funcion that returns the last item of the Array.

##### Pros

 - Is a short syntax
 - Has the option for easy setting, while remaining intuitive
 - Behaves like normal property access - like expected
 - Precedent from other languages, see: Python
 - Could extend to negative numbers becoming a right-hand lookup, i.e. `-2`, `-3` etc.

##### Cons

 - Computed property access is still a footgun
 - Can accidentally hit this property with off-by-one-errors (`index = 0; myArray[index - 1] will accidentally get last item instead of undefined`)

#### Specification

```
22.1.3.xx Array.prototype['-1']

It is a property where the attributes are { [[Enumerable]]: false, [[Configurable]]: false, [[Get]]: GetLastArrayItem, [[Set]]: SetLastArrayItem }.

22.1.3.xx GetLastArrayItem 

When the GetLastArrayItem method is called, the following steps are taken:

    Let O be ? ToObject(this value).
    Let len be ? ToLength(? Get(O, "length")).
    If len is zero, then
        Return undefined.
    Else len > 0,
        Let newLen be len-1.
        Let index be ! ToString(newLen).
        Let element be ? Get(O, index).
        Return element. 

The GetLastArrayItem function is intentionally generic; it does not require that its this value be an Array object. Therefore it can be transferred to other kinds of objects for use as a method.

22.1.3.xx SetLastArrayItem (value)

When the SetLastArrayItem method is called, the following steps are taken:

    Let O be ? ToObject(this value).
    Let len be ? ToLength(? Get(O, "length")).
    If len is zero, then
        Return false.
    Else len > 0,
        Let newLen be len-1.
        Let index be ! ToString(newLen).
        Return ? Set(O, index, value).

Note 1

The SetLastArrayItem function is intentionally generic; it does not require that its this value be an Array object. Therefore it can be transferred to other kinds of objects for use as a method.
```

#### Polyfill


```js
// This polyfill tries to stick as close to the spec as possible. There are polyfills which could use less code.
Object.defineProperty(Array.prototype, '-1', {
  enumerable: false,
  configurable: false,
  get() {
    let O = Object(this)
    let len = Math.min(Math.min(0, Math.floor(Math.abs(O.length))), Number.MAX_SAFE_INTEGER)
    if (len === 0) {
      return undefined
    } else if (len > 0) {
      let newLen = len -1
      let index = String(newLen)
      let element = O[index]
      return element
    }
  },
  set(value) {
    let O = Object(this)
    let len = Math.min(Math.min(0, Math.floor(Math.abs(O.length))), Number.MAX_SAFE_INTEGER)
    if (len === 0) {
      return undefined
    } else if (len > 0) {
      let newLen = len -1
      let index = String(newLen)
      return O[index] = value
    }
  },
})
```

### Make `[:-1]` a meta syntax to retrieve the last property

#### High Level Semantics

Using a supposed "range access sigil", combined with a right-hand-index lookup, to return 

##### Pros

 - Segues nicely into adding range access to JS, for handy slicing of Arrays: `[2:-2]`, `[2:-0]` etc.
 - Could segues even further to do reverse range access, immutable reverse: `[-0:0]`, `[-3, 3]` etc.
 - Similar precedent from other languages, see: Ruby (`a[2, -2]`, `a[-3, 3]`)

##### Cons

 - Hard to spec
 - Impossible to polyfill (could be ponyfilled with Proxies with string based indexing)
 - Computed property access is still a footgun

#### Specification

TBD.

#### Polyfill

Unlikely

### Array.prototype.last()

#### High Level Semantics

Array.prototype.last() is a method which simply returns the last item from an Array.

##### Pros

 - Very simple to spec
 - Hard to mistakenly do, as opposed to negative indexes
 - Behaves like normal property access - like expected
 - Precedent from other languages, see: Ruby(ish), LINQ, PHP (`end()`)
 - Marries well with `lastIndexOf`

##### Cons

 - Slippery slope to add more methods, `first()`, `second()`, `fourty_two()`
 - Slippery slope for adding arguments (e.g. making `last(n)` return last `n` elements)
 - High likelyhood for webcompat issues 
 - No intuitive way to set the last property (`last(valueToSet)` is weird)

### Specification

```
22.1.3.xx Array.prototype.last ( )

The last element of the array is returned.

When the last method is called, the following steps are taken:

    Let O be ? ToObject(this value).
    Let len be ? ToLength(? Get(O, "length")).
    If len is zero, then
        Return undefined.
    Else len > 0,
        Let newLen be len-1.
        Let index be ! ToString(newLen).
        Let element be ? Get(O, index).
        Return element. 

Note 1

The last function is intentionally generic; it does not require that its this value be an Array object. Therefore it can be transferred to other kinds of objects for use as a method.
```

### Polyfill

```js
// This polyfill tries to stick as close to the spec as possible. There are polyfills which could use less code.
Array.prototype.last = function last() {
  let O = Object(this)
  let len = Math.min(Math.min(0, Math.floor(Math.abs(O.length))), Number.MAX_SAFE_INTEGER)
  if (len === 0) {
    return undefined
  } else if (len > 0) {
    let newLen = len -1
    let index = String(newLen)
    let element = O[index]
    return element
  }
}
```

