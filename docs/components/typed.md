# typed — the generics parallel track

`github.com/go-composites/typed` is a **complementary, parallel track** to the
dynamic composites. Where the core repos box values behind `interface{}` and
return `interface{}` payloads, `typed` carries a static type parameter `T`, so
the compiler verifies that producers and consumers agree on the payload type.
A type mismatch that the reflective variant can only surface at run time becomes
a **build error** here.

It is **not a replacement.** Generics erase `T` at instantiation — once a
`Result[int]` is built there is no `int` left in the value to reflect on — so
the dynamic, `interface{}`-based composites remain the more faithful expression
of the org's reflective, message-passing philosophy (`Kind()`, `RespondTo()`,
heterogeneous containers that hold values of different kinds side by side).
Reach for `typed` when **you already know your types and want the compiler's
guarantees**; reach for the dynamic composites when you want runtime
introspection, message passing, or heterogeneity.

Three sub-packages live under `src/`: [`result`](#result),
[`optional`](#optional) and [`slice`](#slice).

## result

```go
import "github.com/go-composites/typed/src/result"

type Result[T any] interface {
    HasError() bool
    Payload() (T, bool)
    Error() error
}

func Ok[T any](v T) Result[T]
func Err[T any](e error) Result[T]

func Map[T, U any](r Result[T], f func(T) U) Result[U]
func FlatMap[T, U any](r Result[T], f func(T) Result[U]) Result[U]
func AndThen[T, U any](r Result[T], f func(T) Result[U]) Result[U]
```

A `Result[T]` is the type-safe railway value: either a success carrying a
statically-typed payload, or a failure carrying an error.

| Member | Behaviour |
|--------|-----------|
| `HasError()` | `true` on the failure branch (defined as `!ok`). |
| `Payload()` | Returns `(value, true)` on success, or `(zero, false)` on failure. |
| `Error()` | Returns the carried error, or `nil` on the success branch. |

Constructors follow the Null-Object spirit: `Ok` and `Err` are **real values,
never nil**. `Err(nil)` normalises a nil error to a non-nil sentinel
(`errors.New("result: nil error")`), so a failure is never silently
indistinguishable from a success and `Error()` never returns `nil` on the
failure branch.

The combinators implement railway-oriented short-circuiting:

- `Map(r, f)` applies `f` to the payload on success, producing a `Result[U]`;
  a failure is propagated unchanged.
- `FlatMap(r, f)` is the monadic bind: `f` is invoked only on success and may
  itself fail; failures short-circuit.
- `AndThen` is a railway-oriented alias for `FlatMap`.

`Map`, `FlatMap` and `AndThen` are free functions (not methods) because a Go
method cannot introduce the second type parameter `U`.

```go
r := result.Ok(2)
doubled := result.Map(r, func(n int) int { return n * 2 }) // Ok(4)

parse := func(n int) result.Result[string] {
    if n < 0 {
        return result.Err[string](errors.New("negative"))
    }
    return result.Ok(strconv.Itoa(n))
}
chained := result.FlatMap(r, parse) // Ok("2")

if v, ok := chained.Payload(); ok {
    fmt.Println(v)
}
```

## optional

```go
import "github.com/go-composites/typed/src/optional"

type Optional[T any] struct{ /* unexported */ }

func Some[T any](v T) Optional[T]
func None[T any]() Optional[T]

func (o Optional[T]) IsPresent() bool
func (o Optional[T]) Get() (T, bool)
func (o Optional[T]) OrElse(fallback T) T

func Map[T, U any](o Optional[T], f func(T) U) Optional[U]
```

`Optional[T]` is the static-typing analogue of the Null-Object pattern: it is
**always a real value, never nil**. Both `Some` and `None` are concrete, so
there is no nil pointer to forget to check.

| Member | Behaviour |
|--------|-----------|
| `IsPresent()` | `true` when a value is held. |
| `Get()` | Returns `(value, true)` when present, else `(zero, false)`. |
| `OrElse(fallback)` | The held value when present, otherwise `fallback`. |
| `Map(o, f)` | `Some(f(v))` when present, else `None[U]()` — a free function so it can introduce `U`. |

```go
o := optional.Some(10)
fmt.Println(o.OrElse(0))                // 10
fmt.Println(optional.None[int]().OrElse(0)) // 0

label := optional.Map(o, func(n int) string {
    return fmt.Sprintf("#%d", n)
}) // Some("#10")
```

## slice

```go
import "github.com/go-composites/typed/src/slice"

type Slice[T any] []T

func Of[T any](items ...T) Slice[T]

func (s Slice[T]) Len() int
func (s Slice[T]) Filter(keep func(T) bool) Slice[T]
func (s Slice[T]) Find(pred func(T) bool) optional.Optional[T]
func (s Slice[T]) Any(pred func(T) bool) bool
func (s Slice[T]) All(pred func(T) bool) bool

func Map[T, U any](s Slice[T], f func(T) U) Slice[U]
func Reduce[T, A any](s Slice[T], initial A, combine func(A, T) A) A
```

`Slice[T]` is a typed `[]T` — the static-typing analogue of the dynamic
[`array`](array.md) (a `[]interface{}` requiring a cast on every read). There is
no `interface{}` in the public API and no runtime type assertion.

| Member | Behaviour |
|--------|-----------|
| `Of(items...)` | Builds a `Slice[T]` from the given elements. |
| `Len()` | Number of elements. |
| `Filter(keep)` | The elements for which `keep` reports `true`. |
| `Find(pred)` | First matching element wrapped in an [`Optional`](#optional); `None` when nothing matches. |
| `Any(pred)` | `true` when at least one element satisfies `pred`. |
| `All(pred)` | `true` when every element satisfies `pred` (vacuously true for an empty slice). |
| `Map(s, f)` | A `Slice[U]` of `f` applied to each element — a free function so it can introduce `U`. |
| `Reduce(s, initial, combine)` | Left-to-right fold from `initial`; `A` is independent of `T`, hence a free function. |

`Find` returns an `optional.Optional[T]` rather than a `(T, bool)` pair so that
"not found" is itself a real Null-Object value, in keeping with the
composition-oriented spirit.

```go
s := slice.Of(1, 2, 3, 4)

evens := s.Filter(func(n int) bool { return n%2 == 0 }) // {2, 4}

doubled := slice.Map(s, func(n int) int { return n * 2 }) // {2, 4, 6, 8}

sum := slice.Reduce(s, 0, func(a, n int) int { return a + n }) // 10

first := s.Find(func(n int) bool { return n > 2 }) // Some(3)
fmt.Println(first.OrElse(-1))                       // 3

fmt.Println(s.Any(func(n int) bool { return n > 3 })) // true
fmt.Println(s.All(func(n int) bool { return n > 0 })) // true
```

## Dependencies

`slice` depends on [`optional`](#optional) (for `Find`'s return type).
`result` and `optional` have no intra-org dependencies beyond the standard
library.
