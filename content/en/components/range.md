---
title: "range — integer interval"
weight: 240
---

# range — integer interval with Result construction {#range-integer-interval-with-result-construction}

`github.com/go-composites/range` (Go package `Range`, imported from the `src/`
sub-directory) is a numeric **interval composite** over Go `int64` bounds.
Modelled on Ruby's `Range`, it supports both an inclusive end (Ruby's `1..5`)
and an exclusive end (Ruby's `1...5`), plus a non-zero integer step. Its
**fallible construction** returns a [`result`]({{< relref "result.md" >}}) so an invalid step
(zero) is a value rather than a panic. The elements it exposes are
[`number`]({{< relref "number.md" >}}) values.

`Range` is deliberately **integer-only** — its bounds and step are Go `int64`
values — which keeps element enumeration tractable.

## API {#api}

```go
import Range "github.com/go-composites/range/src"

type Interface interface {
    Begin() Number.Interface
    End() Number.Interface
    Step() Number.Interface
    ExcludesEnd() bool
    Includes(n int64) bool
    Len() int
    IsEmpty() bool
    Each(fn func(Number.Interface) Result.Interface) Result.Interface
    ToArray() Array.Interface
    IsNull() bool
}

type Option func(*data)

func New(begin, end int64, options ...Option) Result.Interface
func WithStep(step int64) Option
func Exclusive() Option
```

- `New(begin, end)` yields a `Result` whose payload is a `Range` over
  `[begin, end]` with step `1` and an inclusive end.
- `New(..., Range.Exclusive())` excludes the end bound (Ruby's `1...5`).
- `New(..., Range.WithStep(2))` sets the step. A **zero step** produces a
  `Result` carrying `Error.New("step cannot be zero")` instead of a payload —
  construction never panics and never returns `nil`.

| Member | Behaviour |
|--------|-----------|
| `Begin()` / `End()` / `Step()` | Return the lower bound, upper bound and step as [`number`]({{< relref "number.md" >}}) values. |
| `ExcludesEnd()` | Reports whether the range excludes its end bound. |
| `Includes(n)` | Reports whether `n` lies between the bounds (honouring inclusive/exclusive end) and is reachable from `begin` by a whole number of steps. |
| `Len()` | Returns the number of elements. |
| `IsEmpty()` | Returns `true` when the range has no elements. |
| `Each(fn)` | Iterates the elements as `Number`s; short-circuits and returns the first error `Result`, otherwise returns a fresh `Result.New()`. |
| `ToArray()` | Materialises the elements into an [`array`]({{< relref "array.md" >}}) of `Number`s. |
| `IsNull()` | Always returns `false` — a concrete `Range` is never null. |

## Usage {#usage}

```go
r := Range.New(1, 5) // 1..5, step 1
if !r.HasError() {
    rng := r.Payload().(Range.Interface)
    fmt.Println(rng.Len())          // 5
    fmt.Println(rng.Includes(3))    // true
    fmt.Println(rng.ExcludesEnd())  // false
}

// Exclusive end and a custom step:
r2 := Range.New(0, 10, Range.WithStep(2)) // 0,2,4,6,8,10

// A zero step is a value, not a panic:
if bad := Range.New(1, 5, Range.WithStep(0)); bad.HasError() {
    fmt.Println(bad.Error().Message()) // step cannot be zero
}
```

{{< callout type="info" >}}
**The Null-Object variant**

`Range` ships its Null-Object in the `src/null` sub-package. It honours the
full `Interface` without ever being `nil`, and `IsNull()` returns `true`.
{{< /callout >}}

## Dependencies {#dependencies}

`range` depends on [`array`]({{< relref "array.md" >}}), [`number`]({{< relref "number.md" >}}),
[`result`]({{< relref "result.md" >}}) and [`error`]({{< relref "error.md" >}}) (and transitively on
[`null`]({{< relref "null.md" >}})).
