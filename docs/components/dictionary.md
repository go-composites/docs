# dictionary — key→value composite

`github.com/go-composites/dictionary` (Go package `Dictionary`, imported from the
`src/` sub-directory) is the org's **map composite** — the sibling of
[`array`](array.md). Keys are arbitrary comparable values; values are arbitrary.
Fallible lookups return a [`result`](result.md) (never `(value, ok)` and never a
bare `nil`), while presence tests return a [`boolean`](boolean.md).

## API

```go
import Dictionary "github.com/go-composites/dictionary/src"

type Interface interface {
    Set(key, value interface{}) Interface
    Get(key interface{}) Result.Interface
    Has(key interface{}) Boolean.Interface
    Delete(key interface{}) Interface
    Len() int
    Keys() Array.Interface
    Values() Array.Interface
    Each(fn func(key, value interface{}) Result.Interface) Result.Interface
    IsNull() bool
}

func New(options ...Option) Interface
func WithPairs(pairs map[interface{}]interface{}) Option
func Null() Interface
```

- `New()` builds an empty dictionary; `New(Dictionary.WithPairs(m))` seeds it
  from an existing Go map.
- `Null()` returns the **Null-Object** dictionary (see below).

| Member | Behaviour |
|--------|-----------|
| `Set(key, value)` | Stores `value` under `key` and returns the receiver, so calls chain. |
| `Get(key)` | On a hit, returns a `Result` whose payload is the value; on a miss, returns a `Result` carrying `Error.New("key not found")`. Never `nil`, never panics. |
| `Has(key)` | Returns a `Boolean.Interface` reporting whether `key` is present. |
| `Delete(key)` | Removes `key` (a no-op when absent) and returns the receiver, so calls chain. |
| `Len()` | Returns the number of stored pairs. |
| `Keys()` | Returns an [`array`](array.md) of the keys, in unspecified (Go map) order. |
| `Values()` | Returns an [`array`](array.md) of the values, in the same unspecified order as `Keys()`. |
| `Each(fn)` | Invokes `fn` per pair; short-circuits and returns the first `Result` whose `HasError()` is `true`, otherwise returns a fresh `Result.New()`. Order is unspecified. |
| `IsNull()` | Returns `false` for a real dictionary, `true` for the `Null()` one. |

## Usage

```go
d := Dictionary.New(Dictionary.WithPairs(map[interface{}]interface{}{
    "one": 1,
}))
d.Set("two", 2)

if r := d.Get("one"); !r.HasError() {
    fmt.Println(r.Payload()) // 1
}

if r := d.Get("missing"); r.HasError() {
    fmt.Println(r.Error().Message()) // key not found
}

fmt.Println(d.Has("two").IsTrue()) // true
fmt.Println(d.Len())               // 2
```

!!! note "The Null-Object variant"
    `Dictionary.Null()` honours the full `Interface` without ever being `nil`:
    mutating methods (`Set`, `Delete`) are no-ops returning the receiver, every
    lookup misses (`Get` returns a `"key not found"` error, `Has` returns
    `Boolean.False()`), `Len()` is `0`, and `IsNull()` returns `true`. Use it
    wherever a "missing" dictionary must still answer the contract.

## Dependencies

`dictionary` depends on [`array`](array.md), [`boolean`](boolean.md),
[`result`](result.md) and [`error`](error.md) (and transitively on
[`null`](null.md)).
