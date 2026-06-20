# result — payload + error wrapper

`github.com/go-composites/result` (Go package `Result`, imported from the `src/`
sub-directory) is the org's **return-value wrapper**. Instead of returning
`(value, error)` Go-style, methods across `go-composites` return a single
`Result.Interface` carrying both a **payload** and an **error**.

## API

```go
import Result "github.com/go-composites/result/src"

type Interface interface {
    Payload() interface{}
    HasError() bool
    Error() Error.Interface
}

func New(options ...Option) Interface
func WithPayload(payload interface{}) Option
func WithError(error Error.Interface) Option
```

A fresh `Result.New()` defaults its payload to [`Null.New()`](null.md) and its
error to the [null error](error.md#null-error-srcnull), so neither accessor ever
returns a bare `nil`.

```go
r := Result.New(Result.WithPayload(42))
r.Payload() // 42
r.Error()   // the null error (IsNull() == true)
```

| Member | Behaviour |
|--------|-----------|
| `Payload()` | Returns the stored payload; defaults to a non-nil `Null` value. |
| `Error()` | Returns the stored error; defaults to the null error. |
| `HasError()` | `true` iff a **real** error is attached, i.e. `!d.error.IsNull()`. |

## `HasError()`

`HasError()` is defined as:

```go
func (d data) HasError() bool {
    return !d.error.IsNull()
}
```

Because the [null error](error.md#null-error-srcnull) reports `IsNull() == true`,
a `Result` with **no** error reports `HasError() == false`, while a `Result`
carrying a **real** error reports `HasError() == true` — exactly what the name
implies.

```go
Result.New().HasError()                                   // false (no error)
Result.New(Result.WithError(Error.New("x"))).HasError()   // true  (has error)
```

!!! note "The idiomatic guard"
    Across the org, callers branch on the result with the natural shape:

    ```go
    if r := s.Split(" "); r.HasError() {
        fmt.Println(r.Error().Message()) // failure path
    } else {
        use(r.Payload())                 // success path
    }
    ```

## Construction

`New` is variadic over functional options:

- `WithPayload(v)` installs `v` as the payload.
- `WithError(e)` installs `e` as the error.

Options are applied in order over the `Null`/null-error defaults, so a
`Result.New()` with no options is a successful, empty result.

## Dependencies

`result` depends on [`error`](error.md) (including the null-error sentinel) and
[`null`](null.md).
