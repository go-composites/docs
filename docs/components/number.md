# number — boxed numeric value with Result arithmetic

`github.com/go-composites/number` (Go package `Number`, imported from the `src/`
sub-directory) boxes a Go `int64` or `float64` behind the org's interface-first
style. Its fallible arithmetic returns a [`result`](result.md) so that failures —
notably division by zero — are **values**, never panics or bare `nil`.

## API

```go
import Number "github.com/go-composites/number/src"

type Interface interface {
    ToGoInt() int64
    ToGoFloat() float64
    ToGoString() string
    IsInt() bool
    IsFloat() bool
    IsNull() bool
    Add(Interface) Result.Interface
    Sub(Interface) Result.Interface
    Mul(Interface) Result.Interface
    Div(Interface) Result.Interface
    Mod(Interface) Result.Interface
    Abs() Result.Interface
    Neg() Result.Interface
    Equal(Interface) Boolean.Interface
    LessThan(Interface) Boolean.Interface
    GreaterThan(Interface) Boolean.Interface
    Inspect() string
}

func New(options ...Option) Interface
func WithInt(value int64) Option
func WithFloat(value float64) Option
```

- `New()` (no option) yields the **integer zero**; `IsInt()` reports `true`.
- `New(Number.WithInt(42))` seeds an integer value.
- `New(Number.WithFloat(3.5))` seeds a floating-point value.

| Member | Behaviour |
|--------|-----------|
| `ToGoInt()` | Returns the value truncated to a Go `int64`. |
| `ToGoFloat()` | Returns the value as a Go `float64`. |
| `ToGoString()` | Integers render without a decimal point; floats use the shortest round-tripping form. |
| `IsInt()` | Returns `true` when the value was built as an integer. |
| `IsFloat()` | Returns `true` when the value is floating-point (the negation of `IsInt()`). |
| `IsNull()` | Always returns `false` — a concrete `Number` is never null. |
| `Add` / `Sub` / `Mul` | Return a `Result` whose payload is a new `Number` carrying the result. The result stays integer only when **both** operands are integers; otherwise it becomes a float. |
| `Div(other)` | Returns a `Result` whose payload is the quotient — **unless** `other` is zero, in which case the `Result` carries an `Error.New("division by zero")` instead of a payload. It never panics and never returns `nil`. |
| `Mod(other)` | Returns a `Result` whose payload is the remainder of the receiver divided by `other`: Go's `%` operator when **both** operands are integers, otherwise `math.Mod`. When `other` is zero the `Result` carries an `Error.New("modulo by zero")` instead of a payload — it never panics and never returns `nil`. The result stays integer only when both operands are integers. |
| `Abs()` | Returns a `Result` whose payload is a new `Number` holding the absolute value, preserving the receiver's integer or floating-point kind. |
| `Neg()` | Returns a `Result` whose payload is a new `Number` holding the negation, preserving the receiver's integer or floating-point kind. |
| `Equal` / `LessThan` / `GreaterThan` | Return a `Boolean.Interface` comparing the boxed numeric values. |
| `Inspect()` | Returns a one-line `<Number:... kind=... value=...>` string. |

## Usage

```go
a := Number.New(Number.WithInt(10))
b := Number.New(Number.WithInt(3))

if r := a.Add(b); !r.HasError() {
    sum := r.Payload().(Number.Interface)
    fmt.Println(sum.ToGoString()) // 13
}

// Division by zero is a value, not a panic:
if r := a.Div(Number.New(Number.WithInt(0))); r.HasError() {
    fmt.Println(r.Error().Message()) // division by zero
}
```

!!! note "`HasError()` means what it says"
    A successful arithmetic op returns a `Result` with **no** error, so
    `r.HasError() == false`; only a real failure (e.g. `Div` by zero) makes
    `r.HasError() == true`. See [`result`](result.md#haserror).

## Dependencies

`number` depends on [`boolean`](boolean.md), [`result`](result.md) and
[`error`](error.md) (and transitively on [`null`](null.md)).
