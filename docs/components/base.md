# base — the reflective root

`github.com/go-composites/base` (Go package `Base`, imported from the `src/`
sub-directory) is the org's **reflective base**: it gives every composite a
common, self-describing root. A value can answer what kind of interface it is,
whether it responds to a named method, and which methods it exposes — all via
reflection, with no hand-written boilerplate.

## API

```go
import Base "github.com/go-composites/base/src"

type Interface interface {
	Kind() string             // the interface kind, e.g. "Vehicle.Interface"
	RespondTo(name string) bool // whether an exported method `name` exists
	Methods() []string        // the value's exported method names
}

func New() Interface
```

`New()` returns a bare `Base.Interface`. Each method has a **package-level twin**
that operates on any value — these are what an embedding type delegates to:

```go
func Kind(instance interface{}) string
func RespondTo(instance interface{}, methodName string) bool
func Methods(instance interface{}) []string
```

`Kind` derives the name from the value's reflected type (package name +
`.Interface`); `RespondTo` checks `reflect.Value.MethodByName`; `Methods`
enumerates the reflected method set.

## Where it is used

`base` is the **scaffolding the other composites embed and reuse**, keeping each
per-repo surface small. A composite embeds `Base.Interface` in its own interface
and delegates the three methods to the package-level twins:

```go
package Vehicle

import Base "github.com/go-composites/base/src"

type Interface interface {
	Base.Interface
	Start() bool
}

type data struct{}

func New() Interface { return &data{} }

func (d data) Kind() string            { return Base.Kind(d) }
func (d data) RespondTo(n string) bool { return Base.RespondTo(d, n) }
func (d data) Methods() []string       { return Base.Methods(d) }
func (d data) Start() bool             { return true }
```

```go
v := Vehicle.New()
v.Kind()             // "Vehicle.Interface"
v.RespondTo("Start") // true
v.Methods()          // ["Kind" "Methods" "RespondTo" "Start"]
```

This is the reflective half of the org's two analyzers: [`respondto`](../analyzers/respondto.md)
checks that callers use this dispatch rather than concrete type assertions.
