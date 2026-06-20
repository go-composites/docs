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
    Length() int
    Concat(Interface) Result.Interface
    Contains(substr string) bool
    Replace(old, new string) Result.Interface
    Upper() Result.Interface
    Lower() Result.Interface
    Trim() Result.Interface
    Equal(Interface) bool
    StartsWith(prefix string) bool
    EndsWith(suffix string) bool
    Format(args ...interface{}) Result.Interface
    IsNull() bool
}

func New(options ...Option) Interface
func WithGoString(value string) Option

// Null-Object variant (honours the full Interface, never nil):
func Null() Interface
```

- `New()` builds an empty string; `New(String.WithGoString("Hello World!"))`
  seeds a value via a functional option.
- `Set(v)` updates the value and returns a result whose payload is the string
  itself.
- `ToGoString()` returns the underlying Go `string`.
- `Split(sep)` splits on `sep` and returns a result whose payload is an
  [`array`](array.md) of `String` values (each split field boxed as a new
  `String`).
- `Length()` returns the number of **runes** (via `utf8.RuneCountInString`), so
  multibyte UTF-8 content is counted correctly — not the byte length.
- `Concat(other)` returns a result whose payload is a **new** `String` equal to
  the receiver followed by `other`.
- `Contains(substr)` returns a Go `bool`: whether `substr` is within the string.
- `Replace(old, new)` returns a result whose payload is a **new** `String` with
  all non-overlapping instances of `old` replaced by `new`.
- `Upper()` / `Lower()` / `Trim()` each return a result whose payload is a
  **new** `String` with, respectively, all characters upper-cased, lower-cased,
  or leading/trailing white space removed.
- `Equal(other)` returns a Go `bool`: whether the receiver and `other` carry the
  same underlying string.
- `StartsWith(prefix)` returns a Go `bool`: whether the string begins with
  `prefix`.
- `EndsWith(suffix)` returns a Go `bool`: whether the string ends with `suffix`.
- `Format(args...)` returns a result whose payload is a **new** `String` equal to
  `fmt.Sprintf` applied with the receiver's value as the format string and
  `args` as the operands.
- `IsNull()` returns `false` for a real string.

!!! note "Null-Object variant"
    `String.Null()` returns the **Null-Object** string: an empty, immutable
    placeholder that honours the full `Interface` without ever being `nil`.
    `ToGoString()` is `""`, predicates (`Contains`, `Equal`) report `false`,
    `Length()` is `0`, value-producing operations return a successful result
    wrapping the null `String` (or, where chaining is meaningful, the receiver),
    and `IsNull()` returns `true`. Use it instead of a `nil`
    `String.Interface`. (`String.Null()` lives in the main `src` package; the
    repo also ships a sibling `NullString` package at
    `github.com/go-composites/string/src/null` providing the same Null-Object via
    its own `New()` constructor.)

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
