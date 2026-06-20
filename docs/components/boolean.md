# boolean — boxed boolean value

`github.com/go-composites/boolean` (Go package `Boolean`, imported from the
`src/` sub-directory) boxes a Go `bool` behind the org's interface-first style,
exposing truthiness as composite methods.

## API

```go
import Boolean "github.com/go-composites/boolean/src"

type Interface interface {
    ToGoBool() bool
    ToGoString() string
    IsTrue() bool
    IsFalse() bool
    Equal(Interface) Interface
    Inspect() Inspect.Interface
}

func New(b bool) Interface
func True() Interface
func False() Interface
```

- `New(b)` wraps the Go `bool` `b`.
- `True()` is shorthand for `New(true)`; `False()` is shorthand for `New(false)`.

| Member | Behaviour |
|--------|-----------|
| `ToGoBool()` | Returns the underlying Go `bool`. |
| `ToGoString()` | Returns the value as a quoted string, e.g. `"true"` / `"false"`. |
| `IsTrue()` | Returns `true` when the boxed value is `true`. |
| `IsFalse()` | Returns `true` when the boxed value is `false` (the negation of `IsTrue()`). |
| `Equal(other)` | Returns a new `Boolean.Interface` boxing whether the two boxed values are equal. |
| `Inspect()` | Returns an `Inspect.Interface` describing the value (type, address, data). |

## Usage

```go
b := Boolean.True()
fmt.Println(b.ToGoBool())   // true
fmt.Println(b.ToGoString()) // "true"

if b.Equal(Boolean.New(true)).IsTrue() {
    fmt.Println("both are true")
}
```

!!! note "Structural truthiness recognised by `array`"
    `IsTrue() bool` is a **structural** signal. [`array`](array.md)'s combinators
    treat any value carrying an `IsTrue() bool` method as truthy, and so accept a
    `Boolean.Interface` directly — matched by its method set rather than by an
    import. This is deliberate: `array` does **not** import `boolean`, which would
    create an `array → boolean → inspect → string → array` import cycle.

## Dependencies

`boolean` depends only on its own `inspect` sub-package
(`github.com/go-composites/boolean/src/inspect`); it pulls in no other composite.
