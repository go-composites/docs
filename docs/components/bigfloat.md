# bigfloat — arbitrary-precision float with Result arithmetic

`github.com/go-composites/bigfloat` (Go package `BigFloat`, imported from the
`src/` sub-directory) is an **arbitrary-precision floating-point** composite over
Go's stdlib `math/big.Float`. Every value carries a **fixed 256-bit mantissa**,
so sums such as `0.1 + 0.2` are exact rather than subject to binary `float64`
rounding (Ruby's `BigDecimal` as the reference). Its fallible arithmetic returns
a [`result`](result.md) so that failures — notably division by zero — are
**values**, never panics or bare `nil`.

It rounds out the numeric tower alongside [`number`](number.md) (boxed
`int64`/`float64`), [`bignumber`](bignumber.md) (unbounded integers),
[`rational`](rational.md) (exact fractions) and [`complex`](complex.md): where
those cover integers, fractions and the complex plane, `bigfloat` carries
**unbounded-precision decimals**.

## API

```go
import BigFloat "github.com/go-composites/bigfloat/src"

type Interface interface {
    ToGoString() string
    ToFloat64() float64
    IsNull() bool
    Add(Interface) Result.Interface
    Sub(Interface) Result.Interface
    Mul(Interface) Result.Interface
    Div(Interface) Result.Interface
    Abs() Result.Interface
    Neg() Result.Interface
    Equal(Interface) bool
    LessThan(Interface) bool
    GreaterThan(Interface) bool
    Inspect() string
}

func FromFloat64(f float64) Interface
func FromString(s string) Result.Interface
func Null() Interface
```

- `FromFloat64(f)` builds a `BigFloat` from a Go `float64`.
- `FromString(s)` parses a **decimal** string at 256 bits of precision and
  returns a `Result`; on success its payload is the `BigFloat`, otherwise it
  carries an `Error.New("invalid float: ...")`. This is how values with more
  significant digits than a `float64` can represent are constructed — the parse
  never panics and never returns `nil`.
- `Null()` returns the **Null-Object** big float (see below).

| Member | Behaviour |
|--------|-----------|
| `ToGoString()` | Returns the shortest decimal representation that round-trips the value (`math/big.Float.Text('g', -1)`). |
| `ToFloat64()` | Returns the value as a Go `float64`. When the value does not fit, the result follows `math/big.Float.Float64` (the nearest `float64`, possibly an infinity), so prefer `ToGoString()` for arbitrary-precision values. |
| `IsNull()` | Returns `false` — a concrete `BigFloat` is never null. |
| `Add` / `Sub` / `Mul` | Return a `Result` whose payload is a new `BigFloat` carrying the sum / difference / product. A fresh `big.Float` at 256 bits backs the payload; operands are never mutated. |
| `Div(other)` | Returns a `Result` whose payload is the quotient — **unless** `other` is zero, in which case the `Result` carries an `Error.New("division by zero")` instead of a payload. It never panics and never returns `nil`. |
| `Abs()` | Returns a `Result` whose payload is a new `BigFloat` holding the absolute value. |
| `Neg()` | Returns a `Result` whose payload is a new `BigFloat` holding the negation. |
| `Equal` / `LessThan` / `GreaterThan` | Return a plain Go `bool` comparing the values. |
| `Inspect()` | Returns a one-line `<BigFloat:... value=...>` string. |

## Usage

```go
a := BigFloat.FromFloat64(0.1)
b := BigFloat.FromFloat64(0.2)

if r := a.Add(b); !r.HasError() {
    sum := r.Payload().(BigFloat.Interface)
    fmt.Println(sum.ToGoString()) // 0.3  (exact, not 0.30000000000000004)
}

// More significant digits than a float64 can hold:
if r := BigFloat.FromString("0.12345678901234567890123456789012345678901234567890"); !r.HasError() {
    x := r.Payload().(BigFloat.Interface)
    fmt.Println(x.ToGoString())
}

// Division by zero is a value, not a panic:
if r := a.Div(BigFloat.FromFloat64(0)); r.HasError() {
    fmt.Println(r.Error().Message()) // division by zero
}
```

!!! note "The Null-Object variant"
    `BigFloat.Null()` honours the full `Interface` without ever being `nil`:
    `ToGoString()` is `""`, `ToFloat64()` is `0`, every arithmetic method returns
    a `Result` carrying a `method-not-implemented` error, `Equal(other)` is
    `other.IsNull()`, `LessThan`/`GreaterThan` are `false`, `Inspect()` is
    `<NullBigFloat>`, and `IsNull()` returns `true`. (`BigFloat.Null()` lives in
    the main `src` package; the repo also ships a sibling `NullBigFloat` package
    at `github.com/go-composites/bigfloat/src/null`.)

## Dependencies

`bigfloat` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
