# set — unordered collection of unique items

`github.com/go-composites/set` (Go package `Set`, imported from the `src/`
sub-directory) is the org's **set composite** — an unordered collection of
unique items and the sibling of [`array`](array.md), with which it shares the
`Each`/`ToArray` grammar. Items are arbitrary **comparable** values (a `Set` is
backed by a Go map, exactly like a [`dictionary`](dictionary.md) key).
Membership tests return a plain `bool`, fallible iteration returns a
[`result`](result.md), and every method honours the **Null-Object** invariant
(never `nil`).

## API

```go
import Set "github.com/go-composites/set/src"

type Interface interface {
    Add(item interface{}) Interface
    Delete(item interface{}) Interface
    Has(item interface{}) bool
    Len() int
    IsEmpty() bool
    Each(fn func(item interface{}) Result.Interface) Result.Interface
    ToArray() Array.Interface
    Union(other Interface) Interface
    Intersection(other Interface) Interface
    Difference(other Interface) Interface
    IsSubset(other Interface) bool
    Equal(other Interface) bool
    IsNull() bool
}

func New(items ...interface{}) Interface
func Null() Interface
```

- `New(items...)` builds a `Set` seeded with the given items (deduplicated).
- `Null()` returns the **Null-Object** set (see below).

| Member | Behaviour |
|--------|-----------|
| `Add(item)` | Inserts `item` (idempotent — a no-op when already present) and returns the receiver, so calls chain. |
| `Delete(item)` | Removes `item` (a no-op when absent) and returns the receiver, so calls chain. |
| `Has(item)` | Returns a plain `bool` reporting whether `item` is a member. |
| `Len()` | Returns the number of items. |
| `IsEmpty()` | Returns `true` when the set has no items. |
| `Each(fn)` | Invokes `fn` per item; short-circuits and returns the first `Result` whose `HasError()` is `true`, otherwise returns a fresh `Result.New()`. Order is unspecified (Go map order). |
| `ToArray()` | Materialises the set into an [`array`](array.md), in unspecified order. |
| `Union(other)` | Returns a new `Set` of every item present in this set or `other` (Ruby's `\|`). |
| `Intersection(other)` | Returns a new `Set` of only the items present in both sets (Ruby's `&`). |
| `Difference(other)` | Returns a new `Set` of the items present here but not in `other` (Ruby's `-`). |
| `IsSubset(other)` | Reports whether every item of this set is also in `other`. |
| `Equal(other)` | Reports whether both sets contain exactly the same items. |
| `IsNull()` | Returns `false` for a real set, `true` for the `Null()` one. |

## Usage

```go
a := Set.New(1, 2, 3)
b := Set.New(2, 3, 4)

a.Add(3) // idempotent — still {1, 2, 3}

fmt.Println(a.Has(2))                 // true
fmt.Println(a.Len())                  // 3
fmt.Println(a.Intersection(b).Len())  // 2  (2 and 3)
fmt.Println(a.Union(b).Len())         // 4  (1, 2, 3, 4)
fmt.Println(a.Difference(b).Len())    // 1  (1)
```

!!! note "The Null-Object variant"
    `Set.Null()` honours the full `Interface` without ever being `nil`:
    mutating methods (`Add`, `Delete`) are no-ops returning the receiver, every
    query is empty/false/zero (`Has` is `false`, `Len()` is `0`, `IsEmpty()` is
    `true`), the set operations return the receiver, `IsSubset` is `true`,
    `Equal(other)` is `other.IsEmpty()`, and `IsNull()` returns `true`.

## Dependencies

`set` depends on [`array`](array.md) and [`result`](result.md) (and
transitively on [`error`](error.md) and [`null`](null.md)).
