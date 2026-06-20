# composites — the whole vocabulary under one import

`github.com/go-composites/composites` is the org's **meta-package**: a thin
facade that re-exports every building block — [`array`](array.md),
[`boolean`](boolean.md), [`dictionary`](dictionary.md), [`error`](error.md),
[`null`](null.md), [`number`](number.md), [`result`](result.md),
[`string`](string.md) — so a caller can reach the whole vocabulary with a
**single import**.

!!! info "Imported from the module root, not from `src/`"
    Every other repo is imported from its `src/` sub-directory (e.g.
    `import Result "github.com/go-composites/result/src"`). `composites` is the
    **one exception**: it lives at the module root and its package name *is*
    `composites`.

    ```go
    import "github.com/go-composites/composites"
    ```

## API

```go
import "github.com/go-composites/composites"

// Composite interface types (aliases).
type (
    Array      = array/src.Interface
    Boolean    = boolean/src.Interface
    Dictionary = dictionary/src.Interface
    Error      = error/src.Interface
    Null       = null/src.Interface
    Number     = number/src.Interface
    Result     = result/src.Interface
    String     = string/src.Interface
)

// Functional-option types (aliases).
type (
    DictionaryOption = dictionary/src.Option
    NumberOption     = number/src.Option
    ResultOption     = result/src.Option
    StringOption     = string/src.Option
)

// Constructors, re-exported as function values.
var (
    NewArray      = array/src.New
    NewBoolean    = boolean/src.New
    True          = boolean/src.True
    False         = boolean/src.False
    NewString     = string/src.New
    NewError      = error/src.New
    NewNull       = null/src.New
    NewNumber     = number/src.New
    NewResult     = result/src.New
    NewDictionary = dictionary/src.New
)

// Options, re-exported as function values.
var (
    WithGoString = string/src.WithGoString
    WithPayload  = result/src.WithPayload
    WithError    = result/src.WithError
    WithInt      = number/src.WithInt
    WithFloat    = number/src.WithFloat
    WithPairs    = dictionary/src.WithPairs
)
```

*(The aliases above are written `pkg/src.X` for readability; in the source each
is a Go type alias to the corresponding `…src.Interface` / `…src.Option`, and
each constructor is the underlying `…src.New` function value.)*

## Aliases mean zero wrapping

Because the types are **type aliases** and not new named types, a
`composites.Result` *is literally* a `result/src.Interface` — the same type, not
a wrapper around it. There is no boxing, no conversion, and no runtime cost.

This means full interoperability in both directions: a value produced via the
single `composites` import can be passed to any function written against the
individual `…/src` packages, and vice versa. You can adopt `composites` for
convenience in one file and import the granular repos in another, and the two
styles compose without a single cast.

| Member | What it is |
|--------|------------|
| `Array`, `Boolean`, `Dictionary`, `Error`, `Null`, `Number`, `Result`, `String` | Type aliases to each repo's `Interface`. |
| `DictionaryOption`, `NumberOption`, `ResultOption`, `StringOption` | Type aliases to each repo's `Option`. |
| `NewArray`, `NewBoolean`, `NewString`, `NewError`, `NewNull`, `NewNumber`, `NewResult`, `NewDictionary` | The corresponding `New` constructors, re-exported as values. |
| `True`, `False` | The boolean singletons. |
| `WithGoString`, `WithPayload`, `WithError`, `WithInt`, `WithFloat`, `WithPairs` | Functional options, re-exported as values. |

## Usage

A complete program reaching numbers, dictionaries, arrays and results through
the one import (see `examples/demo/main.go`):

```go
package main

import (
    "fmt"

    "github.com/go-composites/composites"
)

func main() {
    div := composites.NewNumber(composites.WithInt(10)).
        Div(composites.NewNumber(composites.WithInt(0)))
    fmt.Printf("10 / 0 → HasError=%t (%v)\n",
        div.HasError(), div.Error().Message())

    d := composites.NewDictionary().Set("k", "v")
    fmt.Printf("dict.Get(k) → %v\n", d.Get("k").Payload())

    a := composites.NewArray()
    a.Push(composites.True())
    fmt.Printf("array.First() → %v\n", a.First().Payload())

    r := composites.NewResult(composites.WithPayload("ok"))
    fmt.Printf("result → %v (err=%t)\n", r.Payload(), r.HasError())
}
```

Because `composites.Result` and `result/src.Interface` are the same type, the
`r` above can be handed directly to any helper declared as
`func(result/src.Interface)` — no adapter required.

## Dependencies

`composites` depends on **all** of the individual repos it re-exports:
[`array`](array.md), [`boolean`](boolean.md), [`dictionary`](dictionary.md),
[`error`](error.md), [`null`](null.md), [`number`](number.md),
[`result`](result.md) and [`string`](string.md).
