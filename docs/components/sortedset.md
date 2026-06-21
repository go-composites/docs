# sortedset — comparator-sorted collection of unique items

`github.com/go-composites/sortedset` (Go package `SortedSet`, imported from the
`src/` sub-directory) is the org's **comparator-ordered set** — a collection of
unique items kept permanently sorted by a caller-supplied `less` comparator
(TreeSet-like), and the sibling of [`set`](set.md) (unordered) and
[`orderedset`](orderedset.md) (first-insertion order). It shares the
`Each`/`ToArray` grammar with [`array`](array.md): iteration, `ToArray` and the
set-algebra results are all emitted in **ascending comparator order** rather than
Go's unspecified map order. Membership tests return a plain `bool`, fallible
iteration and `First`/`Last` return a [`result`](result.md), and every method
honours the **Null-Object** invariant (never `nil`).

Items must satisfy two constraints, which the caller guarantees: they are
**comparable** (the set is backed by a Go map for O(1) membership and dedup,
exactly like a [`dictionary`](dictionary.md) key) **and** orderable by the `less`
comparator passed to `New`.

## API

```go
import SortedSet "github.com/go-composites/sortedset/src"

type Interface interface {
    Add(item interface{}) Interface
    Delete(item interface{}) Interface
    Has(item interface{}) bool
    Len() int
    IsEmpty() bool
    Each(fn func(item interface{}) Result.Interface) Result.Interface
    ToArray() Array.Interface
    First() Result.Interface
    Last() Result.Interface
    Union(other Interface) Interface
    Intersection(other Interface) Interface
    Difference(other Interface) Interface
    IsSubset(other Interface) bool
    Equal(other Interface) bool
    IsNull() bool
}

func New(less func(a, b interface{}) bool, items ...interface{}) Interface
func Null() Interface
```

- `New(less, items...)` builds a `SortedSet` ordered by the `less` comparator and
  seeded with the given items (deduplicated and inserted in sorted position).
- `Null()` returns the **Null-Object** set (see below).

| Member | Behaviour |
|--------|-----------|
| `Add(item)` | Inserts `item` in its sorted position when new (idempotent — a no-op when already present, which keeps the set sorted) and returns the receiver, so calls chain. |
| `Delete(item)` | Removes `item` (a no-op when absent), keeping the rest sorted, and returns the receiver, so calls chain. |
| `Has(item)` | Returns a plain `bool` reporting whether `item` is a member. |
| `Len()` | Returns the number of items. |
| `IsEmpty()` | Returns `true` when the set has no items. |
| `Each(fn)` | Invokes `fn` per item in **ascending comparator order**; short-circuits and returns the first `Result` whose `HasError()` is `true`, otherwise returns a fresh `Result.New()`. |
| `ToArray()` | Materialises the set into an [`array`](array.md), in ascending comparator order. |
| `First()` | Returns a `Result` whose payload is the **minimum** item (by the comparator), or an `Error.New("SortedSet is empty")` when the set is empty. |
| `Last()` | Returns a `Result` whose payload is the **maximum** item (by the comparator), or an `Error.New("SortedSet is empty")` when the set is empty. |
| `Union(other)` | Returns a new `SortedSet` of every item present in this set or `other` (Ruby's `\|`). The result uses the **receiver's** comparator and stays sorted. |
| `Intersection(other)` | Returns a new `SortedSet` of only the items present in both sets (Ruby's `&`), using the receiver's comparator. |
| `Difference(other)` | Returns a new `SortedSet` of the items present here but not in `other` (Ruby's `-`), using the receiver's comparator. |
| `IsSubset(other)` | Reports whether every item of this set is also in `other`. |
| `Equal(other)` | Reports whether both sets hold the same members. Equality is **order-insensitive**: two `SortedSet`s are equal when they hold the same items regardless of their comparators, exactly like a mathematical set. |
| `IsNull()` | Returns `false` for a real set, `true` for the `Null()` one. |

## Usage

```go
less := func(a, b interface{}) bool { return a.(int) < b.(int) }

a := SortedSet.New(less, 3, 1, 2, 1) // {1, 2, 3} — deduplicated and sorted
b := SortedSet.New(less, 2, 3, 4)

fmt.Println(a.Has(2))                // true
fmt.Println(a.Len())                 // 3
fmt.Println(a.ToArray())             // iterates 1, 2, 3 in sorted order
fmt.Println(a.Intersection(b).Len()) // 2  (2 and 3)
fmt.Println(a.Union(b).Len())        // 4  (1, 2, 3, 4)
fmt.Println(a.Difference(b).Len())   // 1  (1)

if r := a.First(); !r.HasError() {
    fmt.Println(r.Payload()) // 1  (the minimum)
}
```

!!! note "The Null-Object variant"
    `SortedSet.Null()` honours the full `Interface` without ever being `nil`:
    mutating methods (`Add`, `Delete`) are no-ops returning the receiver, every
    query is empty/false/zero (`Has` is `false`, `Len()` is `0`, `IsEmpty()` is
    `true`), `Each` is a clean pass, the set operations return the receiver,
    `First`/`Last` return an `Error.New("SortedSet is empty")` `Result`,
    `IsSubset` is `true`, `Equal(other)` is `other.IsEmpty()`, and `IsNull()`
    returns `true`.

## Dependencies

`sortedset` depends on [`array`](array.md) and [`result`](result.md) (and
transitively on [`error`](error.md) and [`null`](null.md)).
