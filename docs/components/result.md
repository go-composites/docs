# result — payload + error wrapper

`github.com/golang-oop/result` (Go package `Result`, imported from the `src/`
sub-directory) is the org's **return-value wrapper**. Instead of returning
`(value, error)` Go-style, methods across `golang-cop` return a single
`Result.Interface` carrying both a **payload** and an **error**.

## API

```go
import Result "github.com/golang-oop/result/src"

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

## `HasError()` — inverted semantics

!!! danger "Known bug: `HasError()` is inverted in the published module"
    In the released module
    (`github.com/golang-oop/result@v0.0.0-20240527141542-cb3939bd2b03`, the
    version the rest of the org links against), `HasError()` is defined as:

    ```go
    func (d data) HasError() bool {
        return d.error.IsNull()
    }
    ```

    Because the **null error** reports `IsNull() == true`, this means a
    `Result` with **no** error reports `HasError() == true`, and a `Result`
    carrying a **real** error reports `HasError() == false` — the **opposite**
    of what the name implies.

    ```go
    Result.New().HasError()                                   // true  (no error!)
    Result.New(Result.WithError(Error.New("x"))).HasError()   // false (has error!)
    ```

This is not a documentation artefact — it is the behaviour downstream code
relies on. The `array` package's tests explicitly note it: a fresh
`Result.New()` is used as the *error-reporting* result that short-circuits
iteration, while a result carrying an `Error.New("sentinel")` is used as the
*ok* result that lets iteration continue. See
[`array`](array.md#each-and-the-inverted-result).

!!! note "Intended fix lives on `main`, but is not the released behaviour"
    The current source on the `main` branch negates the expression
    (`return !d.error.IsNull()`), which would make `HasError()` correct. That
    change has **not** been published as a module version, so consumers pinned
    to the pseudo-version above still get the inverted behaviour. This page
    documents the behaviour you actually get today; do not assume the negated
    form unless you bump the dependency.

## Demo

The repository ships a `main.go` that dumps a default result and prints
`r.HasError()` — which, per the above, prints `true` for an error-free result.
