# array — interface-first slice

`github.com/go-composites/array` (Go package `Array`, imported from the `src/`
sub-directory) is an **interface-first** wrapper around a `[]interface{}`. Every
method returns a [`result`](result.md), so it composes with the rest of the org,
and each method has a package-level twin that operates on a plain
`[]interface{}` without constructing an `Array`.

## API

```go
import Array "github.com/go-composites/array/src"

type Interface interface {
    Each(fn func(int, interface{}) Result.Interface) Result.Interface
    Map(fn func(int, interface{}) Result.Interface) Result.Interface
    Filter(pred func(int, interface{}) Result.Interface) Result.Interface
    Reduce(seed interface{}, fn func(acc, item interface{}) Result.Interface) Result.Interface
    Find(pred func(int, interface{}) Result.Interface) Result.Interface
    Any(pred func(int, interface{}) Result.Interface) Result.Interface
    All(pred func(int, interface{}) Result.Interface) Result.Interface
    Push(interface{}) Result.Interface
    Pop() Result.Interface
    First() Result.Interface
    Fetch(int) Result.Interface
    Last() Result.Interface
    Clear() Result.Interface
    Copy() Result.Interface
}

func New() Interface

// Package-level twins over a raw []interface{}:
func Each(items []interface{}, fn func(int, interface{}) Result.Interface) Result.Interface
func Map(items []interface{}, fn func(int, interface{}) Result.Interface) Result.Interface
func Filter(items []interface{}, pred func(int, interface{}) Result.Interface) Result.Interface
func Reduce(items []interface{}, seed interface{}, fn func(acc, item interface{}) Result.Interface) Result.Interface
func Find(items []interface{}, pred func(int, interface{}) Result.Interface) Result.Interface
func Any(items []interface{}, pred func(int, interface{}) Result.Interface) Result.Interface
func All(items []interface{}, pred func(int, interface{}) Result.Interface) Result.Interface
```

## Accessors and mutators

| Method | Behaviour |
|--------|-----------|
| `Push(x)` | Appends `x`; returns a result whose payload is the array itself. |
| `Pop()` | Removes and returns the **last** element as the result payload. Panics if the array is empty. |
| `First()` | Returns element `0` as the payload. Panics if empty. |
| `Fetch(i)` | Returns element `i` as the payload. Panics if `i` is out of range. |
| `Last()` | Returns the **last** element (`d.value[len-1]`) as the payload. Panics if empty. |
| `Clear()` | Resets to an empty slice; returns an empty result. |
| `Copy()` | Returns an empty result — **not implemented** (no copy is performed). |

!!! note "Bounds are not guarded"
    `Pop`, `First`, `Fetch`, and `Last` index the underlying slice directly and
    will **panic** on an out-of-range access (e.g. on an empty array). They do
    not convert the panic into an error result.

## Iteration and combinators

All combinators take callbacks that return a [`result`](result.md) and
**short-circuit on the first result whose `HasError() == true`** (a real error),
returning that error result unchanged. On a full, error-free pass each returns a
result describing the outcome.

| Method | Behaviour |
|--------|-----------|
| `Each(fn)` | Calls `fn(index, item)` for its side effects. Returns the first error result, or an empty result. |
| `Map(fn)` | Collects each callback's payload into a **new `Array`**. Payload is the new `Array.Interface`. |
| `Filter(pred)` | Keeps items whose `pred` payload is **truthy**, into a new `Array`. Payload is the new `Array.Interface`. |
| `Reduce(seed, fn)` | Left fold threading an accumulator (`fn(acc, item)`); payload is the final accumulator. |
| `Find(pred)` | First item whose `pred` payload is truthy (as payload). No match → a result carrying an `Error`. |
| `Any(pred)` | Payload is a Go `bool`: `true` on the first truthy item, else `false`. Empty array → `false`. |
| `All(pred)` | Payload is a Go `bool`: `false` on the first falsy item, else `true`. Empty array → `true` (vacuous). |

### Truthiness

`Filter`, `Find`, `Any`, and `All` decide whether a predicate's payload counts
as a match via `isTruthy`:

- a Go `bool` is its own value;
- any value exposing an `IsTrue() bool` method (for example a
  `Boolean.Interface`) uses that result — this is matched **structurally**, so
  `array` does not import `boolean` (which would form an import cycle);
- `nil` is **falsy**;
- any other non-nil payload is **truthy**.

## Usage

```go
arr := Array.New()
arr.Push(1)
arr.Push(2)
arr.Push(3)

// Map: double every item into a new Array.
doubled := arr.Map(func(_ int, item interface{}) Result.Interface {
    return Result.New(Result.WithPayload(item.(int) * 2))
})
out := doubled.Payload().(Array.Interface) // [2 4 6]

// Reduce: sum the items.
sum := arr.Reduce(0, func(acc, item interface{}) Result.Interface {
    return Result.New(Result.WithPayload(acc.(int) + item.(int)))
})
total := sum.Payload().(int) // 6

// Any / All over a predicate.
hasEven := arr.Any(func(_ int, item interface{}) Result.Interface {
    return Result.New(Result.WithPayload(item.(int)%2 == 0))
})
yes := hasEven.Payload().(bool) // true
```

```go
// Find: first matching item, or a not-found Error.
r := arr.Find(func(_ int, item interface{}) Result.Interface {
    return Result.New(Result.WithPayload(item.(int) > 1))
})
if r.HasError() {
    fmt.Println(r.Error().Message()) // "Array.Find: no matching item"
} else {
    fmt.Println(r.Payload()) // 2
}
```

The package-level twins behave identically over a raw slice, e.g.
`Array.Map(items, fn)` or `Array.Reduce(items, seed, fn)`.

## Dependencies

`array` depends on [`result`](result.md) and [`error`](error.md) (the latter for
`Find`'s not-found sentinel), and transitively on [`null`](null.md).
