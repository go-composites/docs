# bag — counted collection (multiset)

`github.com/go-composites/bag` (Go package `Bag`, imported from the `src/`
sub-directory) is the org's **multiset composite** — a collection of items each
carrying a **multiplicity** (a count), so the same value can be held more than
once. It rounds out the collection family next to [`set`](set.md) (unique),
[`orderedset`](orderedset.md) (insertion-ordered) and [`sortedset`](sortedset.md)
(comparator-sorted), and shares the `Each`/`ToArray` grammar with
[`array`](array.md). Items are arbitrary **comparable** values (a `Bag` is backed
by a Go map, exactly like a [`dictionary`](dictionary.md) key). Membership tests
return a plain `bool`, fallible iteration returns a [`result`](result.md), and
every method honours the **Null-Object** invariant (never `nil`).

## API

```go
import Bag "github.com/go-composites/bag/src"

type Interface interface {
    Add(item interface{}) Interface
    Remove(item interface{}) Interface
    Count(item interface{}) int
    Has(item interface{}) bool
    Len() int
    DistinctLen() int
    IsEmpty() bool
    Each(fn func(item interface{}, count int) Result.Interface) Result.Interface
    ToArray() Array.Interface
    DistinctArray() Array.Interface
    Sum(other Interface) Interface
    Union(other Interface) Interface
    Intersection(other Interface) Interface
    Difference(other Interface) Interface
    Equal(other Interface) bool
    IsNull() bool
}

func New(items ...interface{}) Interface
func Null() Interface
```

- `New(items...)` builds a `Bag` seeded with the given items, counting duplicates
  — so `New(1, 1, 2)` holds `1` with a count of `2` and `2` with a count of `1`.
- `Null()` returns the **Null-Object** bag (see below).

| Member | Behaviour |
|--------|-----------|
| `Add(item)` | Increments `item`'s count by one and returns the receiver, so calls chain. |
| `Remove(item)` | Decrements `item`'s count by one, dropping the item entirely when its count reaches zero (a no-op when absent), and returns the receiver, so calls chain. |
| `Count(item)` | Returns `item`'s multiplicity, or `0` when absent. |
| `Has(item)` | Returns a plain `bool` reporting whether `item` is present (count greater than zero). |
| `Len()` | Returns the **total** size counting multiplicity (the sum of all counts). |
| `DistinctLen()` | Returns the number of **distinct** items, ignoring multiplicity. |
| `IsEmpty()` | Returns `true` when the bag has no items. |
| `Each(fn)` | Invokes `fn` per **distinct** item with that item and its count; short-circuits and returns the first `Result` whose `HasError()` is `true`, otherwise returns a fresh `Result.New()`. Order is unspecified (Go map order). |
| `ToArray()` | Materialises the bag into an [`array`](array.md), **repeating each item by its count**, in unspecified order. |
| `DistinctArray()` | Materialises the **distinct** items into an [`array`](array.md), each appearing once regardless of count, in unspecified order. |
| `Sum(other)` | Returns a new `Bag` whose count for each item is the **sum** of its counts in this bag and in `other` (multiset additive union). |
| `Union(other)` | Returns a new `Bag` whose count for each item is the **maximum** of its counts in this bag and in `other` (multiset union). |
| `Intersection(other)` | Returns a new `Bag` whose count for each item is the **minimum** of its counts in both bags (multiset intersection); items absent from either bag are omitted. |
| `Difference(other)` | Returns a new `Bag` whose count for each item is its count here **minus** its count in `other`, **floored at zero** (multiset difference); items reaching zero are omitted. |
| `Equal(other)` | Reports whether both bags contain exactly the same items with exactly the same counts. |
| `IsNull()` | Returns `false` for a real bag, `true` for the `Null()` one. |

## Usage

```go
a := Bag.New(1, 1, 2)    // 1×2, 2×1
b := Bag.New(1, 2, 2, 3) // 1×1, 2×2, 3×1

a.Add(1) // 1×3, 2×1

fmt.Println(a.Has(2))         // true
fmt.Println(a.Count(1))       // 3
fmt.Println(a.Len())          // 4  (total with multiplicity)
fmt.Println(a.DistinctLen())  // 2  (distinct items)

fmt.Println(a.Sum(b).Count(2))          // 3  (1 + 2)
fmt.Println(a.Union(b).Count(2))        // 2  (max of 1 and 2)
fmt.Println(a.Intersection(b).Count(1)) // 1  (min of 3 and 1)
fmt.Println(a.Difference(b).Count(1))   // 2  (3 − 1)
```

!!! note "The Null-Object variant"
    `Bag.Null()` honours the full `Interface` without ever being `nil`: mutating
    methods (`Add`, `Remove`) are no-ops returning the receiver, every query is
    empty/false/zero (`Has` is `false`, `Count` is `0`, `Len()`/`DistinctLen()`
    are `0`, `IsEmpty()` is `true`), `Each` is a clean pass, the multiset
    operations return the receiver, `Equal(other)` is `other.IsEmpty()`, and
    `IsNull()` returns `true`.

## Dependencies

`bag` depends on [`array`](array.md) and [`result`](result.md) (and transitively
on [`error`](error.md) and [`null`](null.md)).
