# symbol — interned, immutable identifier

`github.com/go-composites/symbol` (Go package `Symbol`, imported from the `src/`
sub-directory) is the org's equivalent of Ruby's `Symbol` (`:name`): an
**interned, immutable identifier**. Two symbols built from the same name are the
**same** underlying instance, so `Symbol.New("x") == Symbol.New("x")` and
identity comparison is meaningful and cheap. The registry is guarded by a
`sync.Mutex`, making interning safe for concurrent callers.

## API

```go
import Symbol "github.com/go-composites/symbol/src"

type Interface interface {
    ToGoString() string
    Name() string
    Equal(Interface) bool
    IsNull() bool
    Inspect() string
}

func New(name string) Interface
func Null() Interface
```

- `New(name)` returns the **interned** symbol for `name`; calling it again with
  the same name yields the same instance.
- `Null()` returns the **Null-Object** symbol (see below).

| Member | Behaviour |
|--------|-----------|
| `ToGoString()` | Returns the symbol's name as a Go string. |
| `Name()` | An alias for `ToGoString()`. |
| `Equal(other)` | Reports whether `other` denotes the same interned name. Because symbols are interned this is equivalent to pointer identity for real symbols; comparing names also makes it robust against the Null-Object (a null symbol is never equal). |
| `IsNull()` | Returns `false` for a real symbol, `true` for the `Null()` one. |
| `Inspect()` | Renders the symbol as the literal `:name`. |

## Usage

```go
a := Symbol.New("status")
b := Symbol.New("status")

fmt.Println(a == b)          // true — interned, same instance
fmt.Println(a.Equal(b))      // true
fmt.Println(a.Name())        // status
fmt.Println(a.Inspect())     // :status
```

!!! note "The Null-Object variant"
    `Symbol.Null()` honours the full `Interface` without ever being `nil`: it
    has an empty name (`ToGoString()`/`Name()` return `""`), is never equal to
    any symbol — including itself — `Inspect()` renders `:null`, and `IsNull()`
    returns `true`.

## Dependencies

`symbol` has **no intra-org dependencies** — it builds only on the Go standard
library (`fmt`, `sync`).
