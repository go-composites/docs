# complex — complex number with Result arithmetic

`github.com/go-composites/complex` (Go package `Complex`, imported from the
`src/` sub-directory) is a **complex-number** composite over Go's `complex128`,
modelling Ruby's `Complex` — a value `a + bi`. Its fallible operation (`Div`)
returns a [`result`](result.md) so that a division by zero is a **value**, never
a panic or bare `nil`.

It completes the numeric tower alongside [`number`](number.md) (Ruby's `Integer`
and `Float`), [`bignumber`](bignumber.md) (Ruby's unbounded `Integer`) and
[`rational`](rational.md) (Ruby's exact `Rational`): where those carry values on
the real line, `complex` carries a point in the complex plane.

## API

```go
import Complex "github.com/go-composites/complex/src"

type Interface interface {
    Real() float64
    Imaginary() float64
    ToGoString() string
    IsNull() bool
    Add(Interface) Result.Interface
    Sub(Interface) Result.Interface
    Mul(Interface) Result.Interface
    Div(Interface) Result.Interface
    Abs() float64
    Conjugate() Interface
    Equal(Interface) bool
}

func New(real, imag float64) Interface
func FromReal(r float64) Interface
```

- `New(real, imag)` builds `a + bi` from its real and imaginary parts.
- `FromReal(r)` builds a `Complex` whose imaginary part is zero (`r + 0i`).

| Member | Behaviour |
|--------|-----------|
| `Real()` | Returns the real part of the `Complex` as a Go `float64`. |
| `Imaginary()` | Returns the imaginary part of the `Complex` as a Go `float64`. |
| `ToGoString()` | Returns the textual representation as `"a+bi"` or `"a-bi"`, choosing the sign of the imaginary part sensibly. |
| `IsNull()` | Returns `false` — a concrete `Complex` is never null. |
| `Add` / `Sub` | Return a `Result` whose payload is a new `Complex` carrying the component-wise sum / difference. |
| `Mul(other)` | Returns a `Result` whose payload is the product, following the rule `(a+bi)(c+di) = (ac-bd) + (ad+bc)i`. |
| `Div(other)` | Returns a `Result` whose payload is the quotient — **unless** `other` is `0+0i`, in which case the `Result` carries an `Error.New("division by zero")` instead of a payload. It never panics, never yields `NaN`/`Inf`, and never returns `nil`. |
| `Abs()` | Returns the magnitude (modulus) `sqrt(a*a + b*b)` as a Go `float64`. |
| `Conjugate()` | Returns the complex conjugate `a-bi` of the receiver. |
| `Equal(other)` | Returns a plain Go `bool` comparing both the real and imaginary parts. |

## Usage

```go
a := Complex.New(3, 4) // 3+4i
b := Complex.New(1, 2) // 1+2i

if r := a.Add(b); !r.HasError() {
    sum := r.Payload().(Complex.Interface)
    fmt.Println(sum.ToGoString()) // 4+6i
}

fmt.Println(a.Abs())              // 5
fmt.Println(a.Conjugate().ToGoString()) // 3-4i

// Division by 0+0i is a value, not a panic:
if r := a.Div(Complex.New(0, 0)); r.HasError() {
    fmt.Println(r.Error().Message()) // division by zero
}
```

## Dependencies

`complex` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
