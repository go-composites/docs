---
title: "time — deterministic instant + duration"
weight: 160
---

# time — deterministic instant + duration span {#time-deterministic-instant-duration-span}

`github.com/go-composites/time` (Go package `Time`, imported from the `src/`
sub-directory) is a composite over an **instant in time**, wrapping Go's stdlib
`time.Time` (Ruby's `Time` as the reference). Its sibling
[`Duration`](#duration) (the `src/duration` sub-package) wraps a **span of
time**. Both are interface-first, their fallible constructors return a
[`result`]({{< relref "result.md" >}}) so failures are values rather than panics, and both ship a
**Null-Object** variant.

{{< callout type="info" >}}
**Deterministic by design — there is no `Now()`**

`Time` is built **only** from explicit values (`FromUnix`, `Parse`); there is
deliberately **no `Now()`**. This keeps behaviour reproducible across
architectures and test runs.
{{< /callout >}}

## Time {#time}

### API {#api}

```go
import Time "github.com/go-composites/time/src"

type Interface interface {
    ToUnix() int64
    Format(layout string) string
    ToGoString() string
    In(name string) Result.Interface
    Zone() string
    UTC() Interface
    Before(Interface) bool
    After(Interface) bool
    Equal(Interface) bool
    Add(Duration.Interface) Result.Interface
    Sub(Interface) Duration.Interface
    IsNull() bool
}

func FromUnix(sec int64) Interface
func Parse(layout, value string) Result.Interface
func Null() Interface
```

- `FromUnix(sec)` builds a `Time` from a Unix timestamp (seconds since the
  epoch), in UTC.
- `Parse(layout, value)` returns a `Result`; on success its payload is the
  `Time`, otherwise it carries an `Error` (the value did not match the layout).
- `Null()` returns the **Null-Object** time (see below).

| Member | Behaviour |
|--------|-----------|
| `ToUnix()` | Returns the instant as a Unix timestamp (seconds). |
| `Format(layout)` | Renders the instant per the given layout (see `time.Time.Format`). |
| `ToGoString()` | Returns the RFC3339 representation. |
| `In(name)` | Returns a `Result` whose payload is a new `Time` denoting the **same instant** in the IANA location `name` (e.g. `"Europe/Paris"`); when `name` is not a known zone the `Result` carries an `Error` instead. Only the location changes, so the underlying instant — and `Before`/`After`/`Equal` — is preserved. Never panics, never returns `nil`. |
| `Zone()` | Returns the abbreviated name of the time zone in effect at the instant (e.g. `"UTC"`, `"CET"`). |
| `UTC()` | Returns a new `Time` denoting the same instant with its location set to UTC. |
| `Before(other)` / `After(other)` | Report whether the receiver is strictly before / after `other`. |
| `Equal(other)` | Reports whether both denote the same instant (compared by Unix seconds). |
| `Add(duration)` | Returns a `Result` whose payload is a new `Time` shifted forward by `duration`. |
| `Sub(other)` | Returns the [`Duration`](#duration) spanning from `other` to the receiver. |
| `IsNull()` | Returns `false` for a real time, `true` for the `Null()` one. |

### Usage {#usage}

```go
t := Time.FromUnix(0) // 1970-01-01T00:00:00Z
fmt.Println(t.ToGoString()) // 1970-01-01T00:00:00Z

r := Time.Parse(time.RFC3339, "2026-06-20T12:00:00Z")
if !r.HasError() {
    parsed := r.Payload().(Time.Interface)
    fmt.Println(parsed.After(t)) // true
}

if shifted := t.Add(Duration.FromSeconds(3600)); !shifted.HasError() {
    fmt.Println(shifted.Payload().(Time.Interface).ToUnix()) // 3600
}

// Relocate to an IANA zone — same instant, different location:
if r := t.In("Europe/Paris"); !r.HasError() {
    paris := r.Payload().(Time.Interface)
    fmt.Println(paris.Zone())      // CET
    fmt.Println(paris.Equal(t))    // true — the underlying instant is unchanged
}
```

{{< callout type="info" >}}
**IANA zones work on every architecture**

`time` embeds the IANA time-zone database (`time/tzdata`), so `In(...)`
resolves zones like `"Europe/Paris"` deterministically on **every
architecture** and inside the toolchain-less CI containers — without relying
on a system tzdata install. Because relocating only changes the location, the
underlying instant is preserved, so `Before`/`After`/`Equal` are unaffected.
{{< /callout >}}

{{< callout type="info" >}}
**The Null-Object Time**

`Time.Null()` denotes the zero instant: `ToUnix()` is `0`, `Format` returns
`""`, `ToGoString()` is `<NullTime>`, `In(...)` carries an
`Error.New("cannot relocate a null Time")`, `Zone()` is `""`, `UTC()` yields
another null `Time`, comparisons are `false`, `Equal` is `true` only against
another null, `Add` yields a null `Time`, `Sub` yields a null `Duration`, and
`IsNull()` returns `true`.
{{< /callout >}}

## Duration {#duration}

The `src/duration` sub-package (Go package `Duration`) wraps Go's
`time.Duration`.

### API {#api_1}

```go
import Duration "github.com/go-composites/time/src/duration"

type Interface interface {
    ToSeconds() int64
    ToGoString() string
    Add(Interface) Interface
    Sub(Interface) Interface
    Equal(Interface) bool
    IsNull() bool
}

func FromSeconds(sec int64) Interface
func Parse(s string) Result.Interface
func Null() Interface
```

- `FromSeconds(sec)` builds a `Duration` from a whole number of seconds.
- `Parse(s)` returns a `Result`; on success its payload is the `Duration`,
  otherwise it carries an `Error` (the input was not a valid Go duration string
  — see `time.ParseDuration`).
- `Null()` returns the **Null-Object** duration.

| Member | Behaviour |
|--------|-----------|
| `ToSeconds()` | Returns the span truncated to a whole number of seconds. |
| `ToGoString()` | Returns the Go textual form (e.g. `1h30m0s`). |
| `Add(other)` / `Sub(other)` | Return a new `Duration` that is the sum / difference of the receiver and `other`. |
| `Equal(other)` | Reports whether both span the same number of seconds. |
| `IsNull()` | Returns `false` for a real duration, `true` for the `Null()` one. |

### Usage {#usage_1}

```go
d := Duration.FromSeconds(90)
fmt.Println(d.ToGoString()) // 1m30s

r := Duration.Parse("1h30m")
if !r.HasError() {
    fmt.Println(r.Payload().(Duration.Interface).ToSeconds()) // 5400
}
```

{{< callout type="info" >}}
**The Null-Object Duration**

`Duration.Null()` spans zero seconds (`ToSeconds()` is `0`), `ToGoString()`
is `<NullDuration>`, its arithmetic returns itself, `Equal` is `true` only
against another null, and `IsNull()` returns `true`.
{{< /callout >}}

## Dependencies {#dependencies}

Both `time` and its `duration` sub-package depend on [`result`]({{< relref "result.md" >}}) and
[`error`]({{< relref "error.md" >}}) (and transitively on [`null`]({{< relref "null.md" >}})). `Time` also depends
on its own `duration` sub-package.
