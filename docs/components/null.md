# null — the null sentinel

`github.com/golang-oop/null` (Go package `Null`, imported from the `src/`
sub-directory) is the org's minimal **null value**. It is a leaf module with no
dependencies.

## API

```go
import Null "github.com/golang-oop/null/src"

type Interface interface{}

func New() Interface
```

`Null.Interface` is the **empty interface** — any value satisfies it. `New()`
returns an internal `*data` struct (which carries no fields).

The concrete value also has a method:

```go
func (d data) IsNul() bool // always returns true
```

!!! note "`IsNul` is unexported in effect"
    `IsNul()` is spelled without the trailing `l` ("IsNul", not "IsNull") and is
    **not part of `Null.Interface`** (which is `interface{}`). Because the
    concrete `data` type is unexported, callers holding a `Null.Interface`
    cannot invoke `IsNul()` without a type assertion to the package-private
    type, which is impossible from outside the package. In practice the value is
    used purely as an opaque non-nil placeholder.

## Where it is used

[`result`](result.md) installs `Null.New()` as the **default payload** of a
freshly built `Result`, so that `Payload()` never returns a bare `nil`:

```go
r := Result.New()
r.Payload() // a non-nil Null value, not nil
```
