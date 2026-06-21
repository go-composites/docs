# rational — exact rational number with Result arithmetic

`github.com/go-composites/rational` (Go package `Rational`, imported from the
`src/` sub-directory) is an **exact rational-number** composite over Go's stdlib
`math/big.Rat` (Ruby's `Rational` as the reference). A fraction is always stored
in **lowest terms** and is exact — `1/3 + 1/6` is exactly `1/2`, with **no
floating-point error**. Its fallible operations (the constructors and `Div`)
return a [`result`](result.md) so that failures — a zero denominator or a
division by zero — are **values**, never panics or bare `nil`.

It extends the numeric tower alongside [`number`](number.md) (Ruby's `Integer`
and `Float`) and [`bignumber`](bignumber.md) (Ruby's unbounded `Integer`): where
those box whole or floating-point values, `rational` carries an **exact
fraction**.

## API

```go
import Rational "github.com/go-composites/rational/src"

type Interface interface {
    Numerator() int64
    Denominator() int64
    ToGoString() string
    ToFloat() float64
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

func FromInts(num, den int64) Result.Interface
func FromString(s string) Result.Interface
func Null() Interface
```

- `FromInts(num, den)` builds a `Rational` from two Go `int64` values, reduced to
  lowest terms, and returns a `Result`; on success its payload is the `Rational`.
  When `den` is `0` the `Result` carries an `Error.New("zero denominator")`
  instead of a payload — the construction never panics and never returns `nil`.
- `FromString(s)` parses a fraction string such as `"3/4"` and returns a
  `Result`; on success its payload is the parsed, reduced `Rational`, otherwise it
  carries an `Error.New("invalid rational: ...")`. The parse never panics and
  never returns `nil`.
- `Null()` returns the **Null-Object** rational (see below).

| Member | Behaviour |
|--------|-----------|
| `Numerator()` | Returns the numerator of the reduced fraction as a Go `int64`. |
| `Denominator()` | Returns the denominator of the reduced fraction as a Go `int64`. |
| `ToGoString()` | Returns the `"num/den"` representation of the reduced fraction. |
| `ToFloat()` | Returns the value as a Go `float64`. This conversion is **lossy**; the exact value is preserved by `ToGoString()` and the arithmetic methods. |
| `IsNull()` | Returns `false` — a concrete `Rational` is never null. |
| `Add` / `Sub` / `Mul` | Return a `Result` whose payload is a new `Rational` carrying the sum / difference / product. A fresh `big.Rat` backs the payload; operands are never mutated. |
| `Div(other)` | Returns a `Result` whose payload is the quotient — **unless** `other` is a zero rational, in which case the `Result` carries an `Error.New("division by zero")` instead of a payload. It never panics and never returns `nil`. |
| `Abs()` | Returns a `Result` whose payload is a new `Rational` holding the absolute value. |
| `Neg()` | Returns a `Result` whose payload is a new `Rational` holding the negation. |
| `Equal` / `LessThan` / `GreaterThan` | Return a plain Go `bool` comparing the rational values. |
| `Inspect()` | Returns a one-line `<Rational:... value=...>` string. |

## Usage

```go
a := Rational.FromString("1/3").Payload().(Rational.Interface)
b := Rational.FromString("1/6").Payload().(Rational.Interface)

// Exactness — 1/3 + 1/6 is exactly 1/2, with no float error:
if r := a.Add(b); !r.HasError() {
    sum := r.Payload().(Rational.Interface)
    fmt.Println(sum.ToGoString())      // 1/2
    half := Rational.FromInts(1, 2).Payload().(Rational.Interface)
    fmt.Println(sum.Equal(half))       // true
}

// A zero denominator is a value, not a panic:
if r := Rational.FromInts(1, 0); r.HasError() {
    fmt.Println(r.Error().Message())   // zero denominator
}

// Division by a zero rational is a value, not a panic:
zero := Rational.FromInts(0, 1).Payload().(Rational.Interface)
if r := a.Div(zero); r.HasError() {
    fmt.Println(r.Error().Message())   // division by zero
}
```

!!! note "The Null-Object variant"
    `Rational.Null()` honours the full `Interface` without ever being `nil`:
    `Numerator()`/`Denominator()` are `0`, `ToGoString()` is `""`, `ToFloat()` is
    `0`, every arithmetic method returns a `Result` carrying a
    `method-not-implemented` error, `Equal(other)` is `other.IsNull()`,
    `LessThan`/`GreaterThan` are `false`, `Inspect()` is `<NullRational>`, and
    `IsNull()` returns `true`.

## Dependencies

`rational` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
