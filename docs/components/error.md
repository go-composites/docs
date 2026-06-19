# error — error interface + sentinels

`github.com/golang-oop/error` (Go package `Error`, imported from the `src/`
sub-directory) defines the family's **error contract** and a couple of sentinel
implementations. It is a leaf module with no intra-org dependencies.

## The `Error` interface

```go
import Error "github.com/golang-oop/error/src"

type Interface interface {
    Message() string
    IsNull() bool
}
```

`Error.New(message string)` returns a concrete error whose `Message()` returns
the supplied text and whose `IsNull()` returns **`false`** (it is a real error).

```go
e := Error.New("boom")
e.Message() // "boom"
e.IsNull()  // false
```

## Sentinels

### Null error — `src/null`

```go
import NullError "github.com/golang-oop/error/src/null"

e := NullError.New()
e.Message() // "" (empty)
e.IsNull()  // true
```

The null error represents "no error". It is what [`result`](result.md) installs
as the default error of a freshly constructed `Result`.

### Method-not-implemented error — `src/method_not_implemented`

```go
import MethodNotImplementedError "github.com/golang-oop/error/src/method_not_implemented"

e := MethodNotImplementedError.New("Copy")
e.Message() // "The method Copy is not implemented"
e.IsNull()  // false
```

A formatted error for stubbed methods. `New(name)` builds the message
`"The method <name> is not implemented"`.

## Notes

- `IsNull()` is the discriminator used elsewhere in the org to tell a real
  error apart from the "no error" sentinel. The way [`result`](result.md)
  consumes it is the source of the inverted-`HasError()` behaviour documented on
  that page.
