# Changelog

## 12.0.0

`L.forEach` was optimized.

Support for lazy algebras (folds) was removed.

`L.cache` was removed.

`L.findHint` was merged into `L.hint`.

## 11.17.0

Fixed a bug in `L.countIf`.  Previously it didn't pass the index to the
predicate as specified in the documentation.

## 11.16.1

It is now guaranteed that when traversals in this library build intermediate
lists of results and use `of` to create the initial empty list, the value used
for the empty list is a unique value not otherwise exported outside of this
library.  In particular, the values `0`, `null`, `undefined`, `false`, `NaN`,
and `""` are not used as an empty list.  This has the benefit that it is then
possible for client code to give such values special meaning in algebras.

## 11.0.0

Switched the order of arguments to optics so that the first two arguments are
now the same as for an ordinary "read-only" function:

```diff
- (C, xi2yC, x, i) => ...
+ (x, i, C, xi2yC) => ...
```

This way it is not necessary to distinguish between optics and read-only
functions in the `get` operation.  On V8 based JavaScript engines this gives a
significant performance improvement in some operations as taking the `length` of
a function is very expensive in V8.  This also means that the behavior of
composing optics and ordinary functions is different in the sense that more
arguments may be passed to an ordinary function.  This change should only affect
a very small number of users who have written new optics directly against the
internal encoding.  In such a case, you will need to switch the order of
arguments as shown in the above diff.  Also, if you compose optics with ordinary
functions that may use more than two arguments, then you will need to limit
those functions two arguments.

## 10.2.0

Redesigned the experimental `L.findHint`.  The main lesson in the redesign is
that the internally allocated local state for the `hint` was changed to be
explicitly allocated by the caller.  This allows the caller to update the hint
and allows the search to be eliminated in more cases.

## 10.1.1

Previously `L.append` didn't provide its own index.  Now it produces an index
that is the length of the focused array-like object or 0.  This is considered a
bug fix as the behaviour wasn't previously strictly specified.

## 10.0.0

As discussed in
issue [50](https://github.com/calmm-js/partial.lenses/issues/50), to strongly
guide away from mutating data structures, optics
now
[`Object.freeze`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze) any
new objects they create when `NODE_ENV` is not `production`.  Note that optics
do not implicitly "deep freeze" data structures given to them or freeze data
returned by user defined functions.  Only objects newly created by optic
functions themselves are frozen.

Removed previously obsoleted exports:

* `L.firstAs`,
* `L.first`,
* `L.just`,
* `L.mergeAs`,
* `L.merge`, and
* `L.to`.

See previous changelog entries on how you should deal with those.

Lazy folds are no longer considered experimental.  Note, however, that the
technique, an optional `delay` function, upon which they are based is currently
not included in the [Static Land](https://github.com/rpominov/static-land)
specification.  See
issue [40](https://github.com/rpominov/static-land/issues/40) for discussion.

## 9.8.0

Renamed experimental `L.first` and `L.firstAs` as follows:

```diff
-L.first
+L.select

-L.firstAs
+L.selectAs
```

This was done to avoid confusing the operations on traversals with the newly
added `L.last` lens on array-like objects.

## 9.3.0

Obsoleted `L.to` and `L.just`.  You can now directly compose optics with
ordinary functions (whose arity is not 4) and the result is a read-only optic.
This makes `L.to` the same as `R.identity` and `L.just` is the same as
`R.always`.

## 9.1.0

Obsoleted `L.mergeAs` and `L.merge`.  `L.concatAs` and `L.concat` are now just
as fast, so there is no need to have both.

## 9.0.0

`L.augment`, `L.pick` and `L.props` can now be written with an `instanceof
Object` or `undefined`.  Other values are considered errors.  Previously they
could be written with anything, but only a plain `Object` was considered
different from `undefined`.

`L.slice` and `L.filter` can now be written with an array-like object or
`undefined`.  Other values are considered errors.  This was the case earlier
already, but now it is asserted in non-production builds.

`L.index` no longer produces `null` for previously undefined elements.
`L.index` was changed in 4.0.0 to produce `null` elements.  In 8.0.0 treatment
of array-like objects was relaxed, but array producing optics did not
consistently produce `null` elements.  With the relaxed semantics it seems that
producing `null` values would complicate treatment of arrays, so it seems best
to just consistently produce arrays with `undefined` for previously undefined
elements.

## 8.0.0

Relaxed treatment of objects and array like objects.  Previously various optics
required objects to have either `Object` or `Array` as the constructor.  Now any
`instanceof Object` is allowed where previously `Object` constructor was
required and a `String` or an `Object` with non-negative integer `length` is
allowed where previously `Array` constructor was required.  This addresses
issue [40](https://github.com/calmm-js/partial.lenses/issues/40).  See the
documentation of `L.prop` and `L.index` for more details.  The `L.branch`,
`L.elems` and `L.values` traversals have similarly relaxed treatment.

The previously deprecated `L.sequence` traversal was removed.  You need to
explicitly choose either `L.elems` or `L.values`.

Previously undocumented, but accidentally tested for behavior of index lenses to
allow negative indices was removed.  The old behavior was to ignore negative
indices.  The new behavior is to throw an `Error` in non-`production` builds.
Behaviour in `production` builds is undefined.

Removed deprecated `foldMapOf` and `collectMap`.  Use `concatAs` and `collectAs`
instead.

## 7.4.0

Index lenses previously supported using negative indices so that writing through
a negative index was effectively a no-op.  This behavior will not be supported
in the next major version.

## 7.3.0

Deprecated `L.sequence` and introduced `L.elems`, which operates on arrays, and,
`L.values`, which operates on objects, to be used instead.  `L.sequence`
originally only operated on arrays, but it was generalized to operate on objects
in 6.0.0.  Unfortunately that turned out to be a mistake, because in the next
major version, 8.0.0, the plan is to relax the treatment of objects and array
like objects.  The problem is that, with the generalized semantics, the type of
the result, object or array, when writing through `L.sequence` would depend on
the input in an uncontrollable manner.  Apologies for the inconvenience!

## 7.0.0

Added minimal support for indexing.  Various operations and combinators now
provide an index value, either a number for an array index, or a string for an
object property, or undefined in case there is no meaningful index, for the
immediate index being addressed to the user-defined function taken by the
operation or combinator.

## 6.0.0

Removed `L.fromArrayBy`.  It was introduced as an experiment, but the use cases
I had in mind didn't seem to benefit from it.  If you need it, you can use this:

```js
const fromArrayBy = id =>
  iso(xs => {
    if (R.is(Array, xs)) {
      const o = {}, n=xs.length
      for (let i=0; i<n; ++i) {
        const x = xs[i]
        o[x[id]] = x
      }
      return o
    }
  },
  o => R.is(Object, o) ? R.values(o) : undefined)
```

The lens `L.nothing` and the traversal `L.skip` were merged into a single
`L.zero` optic that works like `L.nothing` when being viewed, using `L.get`, and
otherwise like `L.skip`.  The main benefit of this is that it allows "querying"
combinators `L.chain`, `L.choice`, and `L.when` use the one and same `L.zero`
and work without additional glue as traversals.

Generalized the `L.sequence` traversal to also operate on the values of objects.

Removed the defaul import.  The array notation for composition is recommended as
the shorthand of choice.

## 5.3.0

Marked the default import for removal.  With the array shorthand for composition
the default import is no longer worth keeping.

## 5.0.0

Reimplemented library internals
using [Static Land](https://github.com/rpominov/static-land) style dictionaries,
switched to using `infernals` and dropped Ramda dependency and interop.  These
changes were made for the following reasons:

* `infernals` is, and is supposed to remain, a tiny library.  This is an
  advantage if one wishes to use lenses, but does not wish to use Ramda.

* Performance of traversals, and folds over traversals in particular, is and can
  now be significantly improved, because Static Land does not
  require [wrapping](https://github.com/rpominov/static-land#pros) or boxing
  primitive values.

To interop with Ramda, you can write:

```js
import * as L from "partial.lenses"
import * as R from "ramda"

const fromRamda = ramdaLens => L.lens(R.view(ramdaLens), R.set(ramdaLens))
const toRamda = partialLens => R.lens(L.get(partialLens), L.set(partialLens))
```

## 4.0.0

* Removed previously deprecated functionality: `removeAll`.
* Sparse arrays are no longer supported.

## 3.9.2

Although never explicitly specified in documentation, many of the operations and
combinators were curried using Ramda's `curry`.  Unfortunately Ramda's `curry`
is very slow.  From this version forward partial lenses no longer supports the
special features of Ramda's `curry` like placeholders.

## 3.4.1

Fixed bugs when removing a non-existing property from an object or a
non-existent index from an array.  Previously `L.remove("x", {})` returned `{}`.
Now it returns `undefined` as it was previously documented.  Similarly
`L.remove(index, [])` now returns `undefined` as was documented.

Tightened the semantics of combinators, including `L.index`, `L.filter`,
`L.prop` and `L.augment` (and other combinators whose semantics are defined in
terms of those), that specifically work on objects or arrays.  Previously such
combinators worked asymmetrically when operating on values not in their domain.
Now they consistently treat values that are not in their domain as `undefined`.
For example, `L.get("x", null)` now returns `undefined` (previously `null`) and,
consistently, `L.set("x", 1, null)` now returns `{x: 1}` (previously error).

## 3.4.0

Added minimalistic *experimental* traversal support in the form of the
`sequence` traversal.

## 3.0.0

Dropped *implicit* Ramda compatibility.  To interop with Ramda, one must now
explicitly convert lenses using `L.toRamda` and `L.fromRamda`.  In particular,
`L.compose` no longer necessarily returns a Ramda compatible lens and, in the
future, the implementation may be changed more drastically.  This change was
made, because now a lens returned by `L.compose` can take less memory and it
will also be possible to further optimize the implementation in the future.

Removed deprecated functions `L.view`, `L.over` and `L.firstOf`.

## 2.2.0

Renamed `L.view` and `L.over`:

```diff
-L.view
+L.get
```

```diff
-L.over
+L.modify
```

Calling deprecated functions now results in `console.warn` messages.

## 2.1.0

Deprecated `L.firstOf` and added `L.choice`, `L.nothing` and `L.orElse` that
allows the same (and more) functionality to be expressed more compositionally.

## 2.0.0

Changed from using a single default export to named exports to support dead-code
elimination, aka tree shaking.  A number of combinators were renamed in the
process and the default import is now an alias for `compose` that may help to
keep notation concise.

### Upgrade guide

Now using named exports and default that aliases `compose`:

```diff
-import L from "partial.lenses"
+import P, * as L from "partial.lenses"
```

Module prefix no longer works as `compose`:

```diff
-L(...)
+P(...) or L.compose(...)
```

`default` is a keyword and had to be renamed:

```diff
-L.default
+L.defaults
```

`delete` is a keyword and had to be renamed:

```diff
-L.delete
+L.remove
```

```diff
-L.deleteAll
+L.removeAll
```
