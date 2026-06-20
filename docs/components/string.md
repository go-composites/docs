# string — boxed string value

`github.com/go-composites/string` (Go package `String`, imported from the `src/`
sub-directory) boxes a Go `string` behind the org's interface-first style, with
`result`-returning mutators.

## API

```go
import String "github.com/go-composites/string/src"

type Interface interface {
    Set(string) Result.Interface
    ToGoString() string
    Split(string) Result.Interface
    IsNull() bool
}

func New(options ...Option) Interface
func WithGoString(value string) Option
```

- `New()` builds an empty string; `New(String.WithGoString("Hello World!"))`
  seeds a value via a functional option.
- `Set(v)` updates the value and returns a result whose payload is the string
  itself.
- `ToGoString()` returns the underlying Go `string`.
- `Split(sep)` splits on `sep` and returns a result whose payload is an
  [`array`](array.md) of `String` values (each split field boxed as a new
  `String`).
- `IsNull()` always returns `false`.

## Usage

```go
s := String.New(String.WithGoString("Hello World!"))
fmt.Println(s.ToGoString()) // Hello World!

if r := s.Split(" "); r.HasError() {
    fmt.Println(r.Error().Message())
} else {
    arr := r.Payload().(Array.Interface)
    first := arr.First() // payload is a String boxing "Hello"
    _ = first
}
```

`Split` returns an error-free [`result`](result.md) on success, so the
`if r.HasError()` guard above takes the **else** (success) branch — `HasError()`
is `true` only when the result carries a real error.

## Dependencies

`string` depends on [`array`](array.md) and [`result`](result.md) (and
transitively on [`error`](error.md) and [`null`](null.md)).
