# bignumber — arbitrary-precision integer with Result arithmetic

`github.com/go-composites/bignumber` (Go package `BigNumber`, imported from the
`src/` sub-directory) is an **arbitrary-precision integer** composite over Go's
stdlib `math/big.Int` (Ruby's unbounded `Integer` as the reference). There is no
overflow, only memory. Its fallible arithmetic returns a [`result`](result.md) so
that failures — notably division by zero — are **values**, never panics or bare
`nil`.

It complements [`number`](number.md): where `number` boxes a Go `int64`/`float64`
as the fast path, `bignumber` carries **unbounded precision** for integers that
overflow `int64`.

## API

```go
import BigNumber "github.com/go-composites/bignumber/src"

type Interface interface {
    ToGoString() string
    ToInt64() int64
    IsNull() bool
    Add(Interface) Result.Interface
    Sub(Interface) Result.Interface
    Mul(Interface) Result.Interface
    Div(Interface) Result.Interface
    Mod(Interface) Result.Interface
    Abs() Result.Interface
    Neg() Result.Interface
    Equal(Interface) bool
    LessThan(Interface) bool
    GreaterThan(Interface) bool
    Inspect() string
}

func FromInt64(v int64) Interface
func FromString(s string) Result.Interface
func Null() Interface
```

- `FromInt64(v)` builds a `BigNumber` from a Go `int64`.
- `FromString(s)` parses a **base-10 decimal** string and returns a `Result`; on
  success its payload is the `BigNumber`, otherwise it carries an
  `Error.New("invalid integer: ...")`. This is how values that overflow `int64`
  are constructed — the parse never panics and never returns `nil`.
- `Null()` returns the **Null-Object** big number (see below).

| Member | Behaviour |
|--------|-----------|
| `ToGoString()` | Returns the base-10 decimal representation of the value. |
| `ToInt64()` | Returns the value as a Go `int64`. When the value does not fit, the result is the low 64 bits (as `math/big.Int.Int64`), so prefer `ToGoString()` for arbitrary-precision values. |
| `IsNull()` | Returns `false` — a concrete `BigNumber` is never null. |
| `Add` / `Sub` / `Mul` | Return a `Result` whose payload is a new `BigNumber` carrying the sum / difference / product. A fresh `big.Int` backs the payload; operands are never mutated. |
| `Div(other)` | Returns a `Result` whose payload is the quotient — **unless** `other` is zero, in which case the `Result` carries an `Error.New("division by zero")` instead of a payload. It never panics and never returns `nil`. |
| `Mod(other)` | Returns a `Result` whose payload is the remainder of the receiver divided by `other` — **unless** `other` is zero, in which case the `Result` carries an `Error.New("modulo by zero")` instead of a payload. It never panics and never returns `nil`. |
| `Abs()` | Returns a `Result` whose payload is a new `BigNumber` holding the absolute value. |
| `Neg()` | Returns a `Result` whose payload is a new `BigNumber` holding the negation. |
| `Equal` / `LessThan` / `GreaterThan` | Return a plain Go `bool` comparing the integer values. |
| `Inspect()` | Returns a one-line `<BigNumber:... value=...>` string. |

## Usage

```go
a := BigNumber.FromInt64(10)
b := BigNumber.FromInt64(3)

if r := a.Add(b); !r.HasError() {
    sum := r.Payload().(BigNumber.Interface)
    fmt.Println(sum.ToGoString()) // 13
}

// Arbitrary precision — values that overflow int64:
if r := BigNumber.FromString("123456789012345678901234567890"); !r.HasError() {
    big := r.Payload().(BigNumber.Interface)
    fmt.Println(big.ToGoString()) // 123456789012345678901234567890
}

// Division by zero is a value, not a panic:
if r := a.Div(BigNumber.FromInt64(0)); r.HasError() {
    fmt.Println(r.Error().Message()) // division by zero
}
```

!!! note "The Null-Object variant"
    `BigNumber.Null()` honours the full `Interface` without ever being `nil`:
    `ToGoString()` is `""`, `ToInt64()` is `0`, every arithmetic method returns a
    `Result` carrying a `method-not-implemented` error, `Equal(other)` is
    `other.IsNull()`, `LessThan`/`GreaterThan` are `false`, `Inspect()` is
    `<NullBigNumber>`, and `IsNull()` returns `true`.

## Dependencies

`bignumber` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
