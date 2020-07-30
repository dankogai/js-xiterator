[![build status](https://secure.travis-ci.org/dankogai/js-xiterable.png)](http://travis-ci.org/dankogai/js-xiterable)

# js-xiterable

Make ES6 Iterators Functional Again

## Synopsis

Suppose we have a generator like this.

```javascript
function* count(n) {
    let i = 0;
    while (i < n) yield i++;
};
```

We make it more functional like this.

```javascript
import {Xiterable} from './xiterable.js';
const xcount = new Xiterable(n => count(n));
const tens = xcount(10);
const odds = tens.filter(v=>v%2).map(v=>v*v);
const zips = tens.zip(odds);
[...tens];  // [ 0,      1,      2,       3,       4, 5, 6, 7, 8, 9]
[...odds];  // [ 1,      9,     25,      49,      81]
[...zips];  // [[0, 1], [1, 9], [2, 25], [3, 49], [4, 81]]
```

In other words, this module make any iterables work like `Array`, with `.map`, `.filter` and so on.

### Install

```shell
npm install js-xiterable
```

### Usage

locally

```javascript
import {
  Xiterable,
  xiterable,
  zip,
  zipWith,
  xrange,
  repeat
} from './xiterable.js`;
```

remotely

```javascript
import {Xiterable} from 'https://cdn.jsdelivr.net/npm/js-xiterable@0.1.0/xiterable.min.js';
```

### commonjs (node.js)

use [babel] or [esm].

[babel]: https://babeljs.io
[esm]: https://github.com/standard-things/esm

```shell
% node -r esm
Welcome to Node.js v14.5.0.
Type ".help" for more information.
> import * as _X from './xiterable.js'
undefined
> _X
[Module] {
  Xiterable: [Function: Xiterable],
  isIterable: [Function: isIterable],
  repeat: [Function: repeat],
  xiterable: [Function: xiterable],
  xrange: [Function: xrange],
  zip: [Function: zip],
  zipWith: [Function: zipWith]
}
> [..._X.xrange(10).filter(v=>v%2).map(v=>v*v)]
[ 1, 9, 25, 49, 81 ]
> 
```

## Description

### constructor

from any built-in iterables...

```javascript
new Xiterable([0,1,2,3]);
new Xiterable('0123');
new Xiterable(new Uint8Array([0,1,2,3]))
```

or your custom generator (with no argument)...

```javascript
let it = new Xiterable(function *() {
  let i = 0;
  while (true) yield i++;
});
[...it.take(8)]; // [ 0, 1, 2, 3, 4, 5, 6, 7]
```

A factory function is also exported as `xiterable`.

```javascript
import {xiterable as $x} from 'js-xiterable';
$x('01234567').zip('abcdefgh').map(v=>v.join('')).toArray(); /* [
  '0a', '1b', '2c', '3d', '4e', '5f', '6g', '7h'
] */
```

### Instance Methods

#### `.toArray()`

Returns `[...this]`.

#### `.nth`

`.nth(n)` returns the nth element of `this` if the original itertor has `.nth` or Array-like (can access nth element via `[n]`.  In which case `nth()` is auto-generated).

#### `.map`

`.map(fn, thisArg?)` works just like `Array.prototype.map` except:

* `.map` of this module works with infinite iterables. 

* if `this` is finite with working `.nth`, the resulting iterable is also reversible with `.reversed` and random-accissible via `.nth`.

#### `.filter`

`.filter(fn, thisArg?)` works just like `Array.prototype.filter` except:

* `.filter` of this module works with infinite iterables. 

* unlike `.map` the resulting iterable is always marked infinite  because there is no way to know its length lazily, that is, prior to iteration.

#### `.take`

`.take(n)` returns an iterator that takes first `n` elements of `this`.

#### `.drop`

`.drop(n)` returns the iterator that drops first `n` elements of `this`.

#### `.zip`

`.zip(...args)` zips iterators in the `args`. Static version also [available](#Xiterablezip).

```javascript
[...Xiterable.xrange().zip('abcd')]   // [[0,"a"],[1,"b"],[2,"c"],[3,"d"]]
```

#### `.reversed`

returns an iterator that returns elements in reverse order.  `this` must be finite and random-accesible via `.nth()` or exception is thrown.

### instance methods found in `Array.prototype`

The following methods in `Array.prototype` are supported as follows.   For any method `meth`, `[...iter.meth(arg)]` deeply equals to `[...iter].meth(arg)`.

| method        | available? | Comment |
|:--------------|:----:|:---------|
|[concat]       | ✔︎ |    |
|[copyWithin]   | ❌ | mutating |
|[entries]      | ✔︎ |   |
|[every]        | ✔︎ |   |
|[fill]         | ❌ | mutating ; see [repeat](#xiterablerepeat) |
|[filter]       | ✔︎ | see [filter](#filter) |
|[find]         | ✔︎ |   |
|[findIndex]    | ✔︎ |   |
|[flat]         | ✔︎ |   |
|[flatMap]      | ✔︎ |   |
|[forEach]      | ✔︎ |   |
|[includes]     | ✔︎*| * throws `RangeError` on infinite iterables if the 2nd arg is negative |
|[indexOf]      | ✔︎*| * throws `RangeError` on infinite iterables if the 2nd arg is negative|
|[join]         | ✔︎ |   |
|[keys]         | ✔︎ |   |
|[lastIndexOf]  | ✔︎*| * throws `RangeError` on infinite iterables if the 2nd arg is negative|
|[map]          | ✔︎ | see [map](#map) |
|[pop]          | ❌ | mutating |
|[push]         | ❌ | mutating |
|[reduce]       | ✔︎* | * throws `RangeError` on infinite iterables |
|[reduceRight]  | ✔︎* | * throws `RangeError` on infinite iterables |
|[reverse]      | ❌ | mutating.  See [reversed](#reversed) |
|[shift]        | ❌ | mutating |
|[slice]        | ✔︎* | * throws `RangeError` on infinite iterables if any of the args is negative |
|[some]         | ✔︎ |   |
|[sort]         | ❌ | mutating |
|[splice]       | ❌ | mutating |
|[unshift]      | ❌ | mutating |
|[filter]       | ✔︎ |   |


* Mutating functions (functions that change `this`) are deliberately made unavailable. e.g. `pop`, `push`…

* Functions that need toiterate backwards do not work on infinite iterables.  e.g. `lastIndexOf()`, `reduceRight()`…

[concat]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat
[copyWithin]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/copyWithin
[entries]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/entries
[every]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/every
[fill]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/fill
[filter]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter
[find]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find
[findIndex]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/findIndex
[flat]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flat
[flatMap]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap
[forEach]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach
[includes]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes
[indexOf]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf
[join]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/join
[keys]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/keys
[lastIndexOf]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/lastIndexOf
[map]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map
[pop]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/pop
[push]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/push
[reduce]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce
[reduceRight]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reduceRight
[reverse]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/reverse
[shift]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/shift
[slice]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/slice
[some]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/some
[sort]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort
[splice]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice
[unshift]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/unshift
[values]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/values

### static methods

They are also exported so you can:

```javascript
import {repeat,xrange,zip,zipWith} from 'xiterable.js'
```
Examples below assumes

```javascript
import {Xiterable} from 'xiterable.js'.
```

Examples below assumes

#### `Xiterable.zip`

Zips iterators in the argument.

```javascript
[...Xiterable.zip('0123', 'abcd')]   // [[0,"a"],[1,"b"],[2,"c"],[3,"d"]]
```

#### `Xiterable.zipWith`

Zips iterators and then feed it to the function.

```javascript
[...Xiterable.zipWith((a,b)=>a+b, 'bcdfg', 'aeiou')]    // ["ba","ce","di","fo","gu"]
```

#### `Xiterable.xrange`

`xrange()` as Python 2 (or `range()` of Python 3).

```javascript
for (const i of Xiterable.xrange()){ // infinite stream of 0, 1, ...
    console.log(i)
}
```

```javascript
[...Xiterable.xrange(4)]        // [0, 1, 2, 3]
[...Xiterable.xrange(1,5)]      // [1, 2, 3, 4]
[...Xiterable.xrange(1,5,2)]    // [1, 3] 
```

#### `Xiterable.repeat`

Returns an iterator with all elements are the same.

```javascript
for (const i of Xiterable.repeat('spam')) { // infinite stream of 'spam'
    console.log(i)
}
```

```javascript
[...Xiterable.repeat('spam', 4)] // ['spam', 'spam', 'spam', 'spam']
```
