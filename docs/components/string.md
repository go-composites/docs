# string — boxed string value

`github.com/golang-oop/string` (Go package `String`, imported from the `src/`
sub-directory) boxes a Go `string` behind the org's interface-first style, with
`result`-returning mutators.

## API

```go
import String "github.com/golang-oop/string/src"

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

!!! warning "Beware the inverted `HasError()`"
    `Split` returns a normal (error-free) result on success. Because
    [`result.HasError()` is inverted](result.md#haserror-inverted-semantics) in
    the published modules, the `if r.HasError()` guard above takes the
    **error** branch on a *successful* split. The repository's own `main.go`
    contains exactly this pattern. Treat the control flow accordingly until the
    `result` fix is released.

## Dependencies

`string` depends on [`array`](array.md) and [`result`](result.md) (and
transitively on [`error`](error.md) and [`null`](null.md)).
