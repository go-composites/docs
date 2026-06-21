# date — deterministic calendar date

`github.com/go-composites/date` (Go package `Date`, imported from the `src/`
sub-directory) is a composite over a **calendar date** — a year, month and day
with **no time-of-day component** (Ruby's `Date` as the reference). It is the
pure-calendar complement to [`time`](time.md): where `Time` is an **instant**
(and `Duration` a span), `Date` carries only date semantics. It is
interface-first, its fallible constructors (`FromYMD`, `Parse`, `AddDays`) return
a [`result`](result.md) so failures are values rather than panics, and it ships a
**Null-Object** variant.

!!! note "Deterministic by design — there is no `Today()`"
    `Date` is built **only** from explicit values (`FromYMD`, `Parse`); there is
    deliberately **no `Today()`** (nor `Now()`). This keeps behaviour
    reproducible across architectures and test runs.

!!! note "Backed by stdlib `time`, but date-only"
    Internally a `Date` is a Go `time.Time` pinned to **midnight UTC**. That
    backing is used only for **calendar validation** (`FromYMD` round-trips
    through `time.Date`, so `Feb 30` or month `13` is rejected) and **day
    arithmetic** (`AddDays`/`DaysBetween`); the time-of-day is never exposed, so
    the surface is purely calendar-based.

## API

```go
import Date "github.com/go-composites/date/src"

type Interface interface {
    Year() int
    Month() int
    Day() int
    Weekday() string
    ToGoString() string
    Before(Interface) bool
    After(Interface) bool
    Equal(Interface) bool
    AddDays(n int) Result.Interface
    DaysBetween(Interface) int
    IsNull() bool
}

func FromYMD(year, month, day int) Result.Interface
func Parse(value string) Result.Interface
func Null() Interface
```

- `FromYMD(year, month, day)` returns a `Result`; on success its payload is the
  `Date`, otherwise it carries an `Error.New("invalid calendar date")` — a date
  that normalises to different components (e.g. `Feb 30`, month `13`) is
  rejected. It never panics and never returns `nil`.
- `Parse(value)` builds a `Date` from an ISO `"YYYY-MM-DD"` value; it returns a
  `Result` whose payload is the `Date`, or an `Error` when the value does not
  match the ISO layout.
- `Null()` returns the **Null-Object** date (see below).

| Member | Behaviour |
|--------|-----------|
| `Year()` | Returns the calendar year. |
| `Month()` | Returns the calendar month (`1`–`12`). |
| `Day()` | Returns the day of the month (`1`–`31`). |
| `Weekday()` | Returns the English name of the day of the week (e.g. `"Monday"`). |
| `ToGoString()` | Returns the ISO `"YYYY-MM-DD"` representation. |
| `Before(other)` / `After(other)` | Report whether the receiver falls strictly before / after `other`. |
| `Equal(other)` | Reports whether the receiver and `other` denote the same calendar date. |
| `AddDays(n)` | Returns a `Result` whose payload is a new `Date` `n` days later (`n` may be negative). It never panics and never returns `nil`. |
| `DaysBetween(other)` | Returns the **signed** number of days from the receiver to `other` (positive when `other` is later, negative when earlier). |
| `IsNull()` | Returns `false` for a real date, `true` for the `Null()` one. |

## Usage

```go
r := Date.FromYMD(2024, 2, 29) // valid: 2024 is a leap year
if !r.HasError() {
    d := r.Payload().(Date.Interface)
    fmt.Println(d.ToGoString()) // 2024-02-29
    fmt.Println(d.Weekday())    // Thursday
}

// An impossible calendar date is a value, not a panic:
if bad := Date.FromYMD(2023, 2, 29); bad.HasError() {
    fmt.Println(bad.Error().Message()) // invalid calendar date
}

if p := Date.Parse("2026-06-21"); !p.HasError() {
    start := p.Payload().(Date.Interface)

    if shifted := start.AddDays(10); !shifted.HasError() {
        end := shifted.Payload().(Date.Interface)
        fmt.Println(end.ToGoString())        // 2026-07-01
        fmt.Println(start.Before(end))       // true
        fmt.Println(start.DaysBetween(end))  // 10
    }
}
```

!!! note "The Null-Object Date"
    `Date.Null()` denotes the empty date: `Year()`/`Month()`/`Day()` are `0`,
    `Weekday()` and `ToGoString()` return `""`, its comparisons are `false`,
    `Equal` is `true` only against another null, `AddDays` yields a `Result`
    wrapping another null `Date`, `DaysBetween` is `0`, and `IsNull()` returns
    `true`.

## Dependencies

`date` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
