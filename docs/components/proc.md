# proc — first-class callable with railway composition

`github.com/go-composites/proc` (Go package `Proc`, imported from the `src/`
sub-directory) is a **first-class callable** composite — the org's analogue of
Ruby's `Proc`/lambda. A `Proc` wraps a single function as a value you can pass
around, invoke, and chain. Every `Proc` is a real object: it never returns a
bare `nil`, it answers `IsNull()`, and even the **Null** `Proc` is safe to
`Call`. Chaining with `Then` is **railway-oriented** — it short-circuits on an
error [`result`](result.md).

## API

```go
import Proc "github.com/go-composites/proc/src"

type Interface interface {
    Call(args ...interface{}) Result.Interface
    Then(next Interface) Interface
    IsNull() bool
}

func New(fn func(args ...interface{}) Result.Interface) Interface
func Null() Interface
```

- `New(fn)` wraps `fn` as a `Proc`. If `fn` is `nil` the resulting `Proc` is
  still safe — `Call` returns an error `Result` rather than panicking.
- `Null()` returns the **Null-Object** (not-callable) `Proc` (see below).

| Member | Behaviour |
|--------|-----------|
| `Call(args...)` | Invokes the wrapped function with the given arguments and returns its `Result`. A `Proc` whose wrapped function is `nil` does not panic — it returns an `Error.New("Proc.Call: nil function")` `Result`. |
| `Then(next)` | Returns a new `Proc` that calls the receiver and, on a successful `Result`, feeds that result's payload as the single argument to `next.Call`. If the receiver yields an error `Result`, `Then` short-circuits and returns that error unchanged — `next` is never called. |
| `IsNull()` | Returns `false` for an ordinary `Proc`, `true` for the `Null()` one. |

## Usage

```go
double := Proc.New(func(args ...interface{}) Result.Interface {
    n := args[0].(int)
    return Result.New(Result.WithPayload(n * 2))
})

incr := Proc.New(func(args ...interface{}) Result.Interface {
    n := args[0].(int)
    return Result.New(Result.WithPayload(n + 1))
})

// Railway composition: double then incr.
if r := double.Then(incr).Call(5); !r.HasError() {
    fmt.Println(r.Payload()) // 11
}
```

!!! note "The Null-Object variant"
    `Proc.Null()` is not callable: `Call` returns an
    `Error.New("Proc.Null: not callable")` `Result`, `Then` returns the Null
    `Proc` again (so chains stay on the null), and `IsNull()` returns `true`.

## Dependencies

`proc` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
