# orderedset — insertion-ordered collection of unique items

`github.com/go-composites/orderedset` (Go package `OrderedSet`, imported from the
`src/` sub-directory) is the **insertion-ordered sibling** of [`set`](set.md) — a
collection of unique items that remembers the order in which each item was first
added. It shares `set`'s full API and the `Each`/`ToArray` grammar of
[`array`](array.md), but where `set` iterates in Go's unspecified map order,
`orderedset` iterates — and emits its `ToArray` and set-algebra results — in
**first-insertion order**. Items are arbitrary **comparable** values (it is backed
by a slice for order plus a Go map for O(1) membership, exactly like a
[`dictionary`](dictionary.md) key). Membership tests return a plain `bool`,
fallible iteration returns a [`result`](result.md), and every method honours the
**Null-Object** invariant (never `nil`).

## API

```go
import OrderedSet "github.com/go-composites/orderedset/src"

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

- `New(items...)` builds an `OrderedSet` seeded with the given items
  (deduplicated and kept in first-seen order).
- `Null()` returns the **Null-Object** ordered set (see below).

| Member | Behaviour |
|--------|-----------|
| `Add(item)` | Inserts `item`, appending it to the insertion order only when it is new (a no-op for an already-present item, which **keeps its original position**), and returns the receiver, so calls chain. |
| `Delete(item)` | Removes `item` (a no-op when absent), preserving the relative order of the remaining items, and returns the receiver, so calls chain. |
| `Has(item)` | Returns a plain `bool` reporting whether `item` is a member. |
| `Len()` | Returns the number of items. |
| `IsEmpty()` | Returns `true` when the set has no items. |
| `Each(fn)` | Invokes `fn` per item **in insertion order**; short-circuits and returns the first `Result` whose `HasError()` is `true`, otherwise returns a fresh `Result.New()`. |
| `ToArray()` | Materialises the set into an [`array`](array.md), **in insertion order**. |
| `Union(other)` | Returns a new `OrderedSet` of every item present in this set or `other` (Ruby's `\|`): the receiver's items first in their order, then `other`'s remaining items in `other`'s order. |
| `Intersection(other)` | Returns a new `OrderedSet` of only the items present in both sets (Ruby's `&`), in the receiver's order. |
| `Difference(other)` | Returns a new `OrderedSet` of the items present here but not in `other` (Ruby's `-`), in the receiver's order. |
| `IsSubset(other)` | Reports whether every item of this set is also in `other`. |
| `Equal(other)` | Reports whether both sets contain exactly the same members. Equality is **order-insensitive** — two `OrderedSet`s are equal when they hold the same items regardless of insertion order, exactly like a mathematical set. |
| `IsNull()` | Returns `false` for a real set, `true` for the `Null()` one. |

## Usage

```go
s := OrderedSet.New(3, 1, 2)
s.Add(1) // idempotent — keeps its original position

s.Each(func(item interface{}) Result.Interface {
    fmt.Println(item) // 3, 1, 2 — in insertion order
    return Result.New()
})

fmt.Println(s.ToArray().Len()) // 3

// Equality ignores order:
fmt.Println(OrderedSet.New(1, 2, 3).Equal(OrderedSet.New(3, 2, 1))) // true
```

!!! note "The Null-Object variant"
    `OrderedSet.Null()` honours the full `Interface` without ever being `nil`:
    mutating methods (`Add`, `Delete`) are no-ops returning the receiver, every
    query is empty/false/zero (`Has` is `false`, `Len()` is `0`, `IsEmpty()` is
    `true`), the set operations return the receiver, `IsSubset` is `true`,
    `Equal(other)` is `other.IsEmpty()`, and `IsNull()` returns `true`.

## Dependencies

`orderedset` depends on [`array`](array.md) and [`result`](result.md) (and
transitively on [`error`](error.md) and [`null`](null.md)).
