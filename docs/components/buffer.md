# buffer — a mutable text buffer (StringBuilder)

`github.com/go-composites/buffer` (Go package `Buffer`, imported from the `src/`
sub-directory) is the org's **mutable text-buffer** composite — a StringBuilder
over Go's stdlib `strings.Builder`. It is the **mutable counterpart** to the
immutable [`string`](string.md) value composite: where `string`'s
value-producing operations each return a **new** boxed value, a `Buffer`
accumulates text **in place**. `Append`, `AppendRune` and `Reset` **mutate the
receiver** and return it, so calls chain. It never goes `nil`: `Null()` yields a
**Null-Object** Buffer that honours the full `Interface`.

## API

```go
import Buffer "github.com/go-composites/buffer/src"

type Interface interface {
    Append(s string) Interface
    AppendRune(r rune) Interface
    ToGoString() string
    Len() int
    IsEmpty() bool
    Reset() Interface
    IsNull() bool
}

func New() Interface
func From(s string) Interface
func Null() Interface
```

- `New()` builds a new, empty `Buffer`.
- `From(s)` builds a new `Buffer` seeded with the initial text `s`.
- `Null()` returns the **Null-Object** buffer (see below).

| Member | Behaviour |
|--------|-----------|
| `Append(s)` | Writes `s` to the end of the buffer, **mutating it in place**, and returns the receiver, so calls chain. |
| `AppendRune(r)` | Writes a single rune to the end of the buffer, **mutating it in place**, and returns the receiver, so calls chain. |
| `ToGoString()` | Returns the accumulated text as a Go `string`. |
| `Len()` | Returns the length of the accumulated text in **bytes**. |
| `IsEmpty()` | Returns `true` when the buffer holds no text. |
| `Reset()` | Clears the buffer, **mutating it in place**, and returns the receiver, so calls chain. |
| `IsNull()` | Returns `false` for a real buffer, `true` for the `Null()` one. |

## Usage

```go
b := Buffer.New().
    Append("Hello").
    Append(", ").
    Append("World").
    AppendRune('!')

fmt.Println(b.ToGoString()) // Hello, World!
fmt.Println(b.Len())        // 13
fmt.Println(b.IsEmpty())    // false

b.Reset() // mutates b in place — now empty
fmt.Println(b.IsEmpty())    // true

// From seeds initial text:
c := Buffer.From("seed")
fmt.Println(c.ToGoString()) // seed
```

!!! note "The Null-Object variant"
    `Buffer.Null()` honours the full `Interface` without ever being `nil`: it
    holds no text, so its mutators (`Append`, `AppendRune`, `Reset`) are no-ops
    returning the null buffer, `ToGoString()` is `""`, `Len()` is `0`,
    `IsEmpty()` is `true`, and `IsNull()` returns `true`.

## Dependencies

`buffer` is self-contained — it wraps the stdlib `strings.Builder` and has no
intra-org dependencies.
