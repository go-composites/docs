# pair — fixed two-element heterogeneous grouping

`github.com/go-composites/pair` (Go package `Pair`, imported from the `src/`
sub-directory) is a fixed **two-element heterogeneous grouping** — the natural
key/value entry of Composition-Oriented Programming. The two slots hold
arbitrary values, a `Pair` never goes `nil`, and the **Null-Object** variant
honours the whole `Interface`.

## API

```go
import Pair "github.com/go-composites/pair/src"

type Interface interface {
    First() interface{}
    Second() interface{}
    Equal(Interface) bool
    ToArray() Array.Interface
    ToGoString() string
    IsNull() bool
}

func New(first, second interface{}) Interface
func Null() Interface
```

- `New(first, second)` holds the two values.
- `Null()` returns the **Null-Object** pair (see below).

| Member | Behaviour |
|--------|-----------|
| `First()` | Returns the first slot. |
| `Second()` | Returns the second slot. |
| `Equal(other)` | Reports whether both slots are deeply equal to `other`'s (compared with `reflect.DeepEqual`). A null pair is never equal to anything. |
| `ToArray()` | Returns a two-element [`array`](array.md) holding `[first, second]`. |
| `ToGoString()` | Renders the pair as `(first, second)`. |
| `IsNull()` | Returns `false` for a real pair, `true` for the `Null()` one. |

## Usage

```go
p := Pair.New("one", 1)

fmt.Println(p.First())      // one
fmt.Println(p.Second())     // 1
fmt.Println(p.ToGoString()) // (one, 1)
fmt.Println(p.Equal(Pair.New("one", 1))) // true
fmt.Println(p.ToArray().Len())           // 2
```

!!! note "The Null-Object variant"
    `Pair.Null()` honours the full `Interface` without ever being `nil`: both
    slots are `nil`, `Equal` is always `false` (a null pair equals nothing),
    `ToArray()` returns the empty array, `ToGoString()` renders `(null)`, and
    `IsNull()` returns `true`.

## Dependencies

`pair` depends on [`array`](array.md) (and uses the Go standard library `fmt`
and `reflect`).
