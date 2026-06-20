# enumerator — lazy sequence

`github.com/go-composites/enumerator` (Go package `Enumerator`, imported from the
`src/` sub-directory) is a **lazy** sequence composite in the spirit of Ruby's
`Enumerator::Lazy`. Its transforms (`Map`/`Filter`/`Take`) build on a producer
and evaluate **nothing** until a terminal operation pulls items through — so you
can `Take` a finite prefix of an **infinite** source without it running away.

## API

```go
import Enumerator "github.com/go-composites/enumerator/src"

type Interface interface {
    // Lazy transforms — return a new Enumerator, evaluate nothing.
    Map(fn func(interface{}) interface{}) Interface
    Filter(pred func(interface{}) bool) Interface
    Take(n int) Interface
    // Terminal operations — force evaluation.
    ToArray() Array.Interface
    Each(fn func(interface{}) Result.Interface) Result.Interface
    First() Result.Interface
    Reduce(seed interface{}, fn func(acc, item interface{}) Result.Interface) Result.Interface
    IsNull() bool
}

func New(items ...interface{}) Interface                                    // finite source
func Generate(seed interface{}, fn func(interface{}) interface{}) Interface // infinite source
func Null() Interface                                                       // Null-Object
```

## Behaviour

| Kind | Method | Behaviour |
|------|--------|-----------|
| lazy | `Map(fn)` | yields `fn(item)` for each upstream item |
| lazy | `Filter(pred)` | yields only items where `pred(item)` is true |
| lazy | `Take(n)` | yields at most `n` items, then **stops the upstream producer** (so it terminates an infinite source); `n ≤ 0` is empty |
| terminal | `ToArray()` | materialise into an [`array`](array.md) |
| terminal | `Each(fn)` | run `fn` per item; **short-circuit** on the first error `result` |
| terminal | `First()` | a `result` with the first item; empty → an error `result` (`"Enumerator.First: empty"`) |
| terminal | `Reduce(seed, fn)` | left fold, short-circuiting on the first error `result` |

The two constructors differ in their source: `New(items…)` is finite, while
`Generate(seed, fn)` is **infinite** (`seed, fn(seed), fn(fn(seed)), …`) and only
terminates once a downstream `Take`/`First` stops it.

## Usage

```go
// A finite prefix of an infinite sequence — lazily.
squares := Enumerator.
    Generate(1, func(x interface{}) interface{} { return x.(int) + 1 }).
    Map(func(x interface{}) interface{} { return x.(int) * x.(int) }).
    Take(3).
    ToArray() // payload elements: 1 4 9
```

!!! note "Null-Object"
    `Enumerator.Null()` is the never-nil empty sequence: transforms return it,
    `ToArray()` is empty, `First()` errors, `Each`/`Reduce` are no-op successes
    (the latter returns the seed), and `IsNull()` reports `true`.

## Dependencies

`enumerator` depends on [`array`](array.md) (for `ToArray`),
[`result`](result.md) and [`error`](error.md), and transitively on
[`null`](null.md).
