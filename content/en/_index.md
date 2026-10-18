---
title: "Home"
weight: 10
---

# go-composites {#go-composites}

A collection of **pure-Go composites** for **Composition-Oriented
Programming**: small, interface-first building blocks — a `null` sentinel, an
`error` interface, a `result` wrapper, boxed `boolean`/`number`/`string` values,
an `array` and a `dictionary`, a `compose` pipeline, a one-import meta-package,
and a `typed` generics track. No cgo, no third-party runtime dependencies (only
`go-spew` in a demo `main`).

The design is deliberately composition-oriented: each package exposes an
`Interface` and a private `data` struct constructed through a `New(...)`
function (several use functional options such as `WithPayload` / `WithGoString`).
Composites are assembled by composing and delegating to smaller composites, never
by inheritance. Fallible methods return a [`result`]({{< relref "components/result.md" >}}) value
rather than the Go idiom of multiple return values, so the packages compose with
one another, and the **Null-Object** pattern runs throughout — a fresh `Result`,
`Error` or `Null` is a real, non-nil value, so callers branch on `HasError()`
instead of nil checks.

## Module paths {#module-paths}

{{< callout type="info" >}}
**Org and module path**

These projects live in the GitHub organisation **`go-composites`**, and their
Go module paths match it — **`github.com/go-composites/<name>`**. For example
the `array` repository declares `module github.com/go-composites/array`. The
importable package lives under the `src/` sub-directory (e.g.
`github.com/go-composites/array/src`, package `Array`).

The one exception is the [`composites`]({{< relref "components/composites.md" >}})
**meta-package**, which is imported at the module root —
`import "github.com/go-composites/composites"`.
{{< /callout >}}

## Components {#components}

| Module | Import path | What it does |
|--------|-------------|--------------|
| [`base`]({{< relref "components/base.md" >}})   | `github.com/go-composites/base/src` | The reflective root every composite embeds: `Kind()`/`RespondTo()`/`Methods()` via reflection, so each repo's surface stays small. |
| [`null`]({{< relref "components/null.md" >}})   | `github.com/go-composites/null/src` | A minimal null-object sentinel used as the default `result` payload. |
| [`error`]({{< relref "components/error.md" >}}) | `github.com/go-composites/error/src` | The `error` interface (`Message`/`IsNull`) plus null and `method-not-implemented` sentinels. |
| [`result`]({{< relref "components/result.md" >}}) | `github.com/go-composites/result/src` | Wraps a `payload` plus an `error` built with functional options; `HasError()` is `true` only on a real error. |
| [`boolean`]({{< relref "components/boolean.md" >}}) | `github.com/go-composites/boolean/src` | A boxed boolean (`IsTrue`/`IsFalse`/`Equal`); its structural `IsTrue()` is what `array`'s combinators treat as truthy. |
| [`number`]({{< relref "components/number.md" >}}) | `github.com/go-composites/number/src` | A boxed numeric value (int/float); arithmetic returns a `result`, so division by zero is an error value, not a panic. |
| [`bignumber`]({{< relref "components/bignumber.md" >}}) | `github.com/go-composites/bignumber/src` | An arbitrary-precision integer over `math/big.Int` (no overflow); unbounded-precision complement to `number`, arithmetic returns a `result`. |
| [`rational`]({{< relref "components/rational.md" >}}) | `github.com/go-composites/rational/src` | An exact rational number over `math/big.Rat` (Ruby's `Rational`); fractions stay in lowest terms, arithmetic returns a `result`. |
| [`complex`]({{< relref "components/complex.md" >}}) | `github.com/go-composites/complex/src` | A complex number `a+bi` over `complex128` (Ruby's `Complex`); arithmetic returns a `result`, with `Abs`/`Conjugate`. |
| [`bigfloat`]({{< relref "components/bigfloat.md" >}}) | `github.com/go-composites/bigfloat/src` | An arbitrary-precision float over `math/big.Float` (256-bit mantissa, Ruby's `BigDecimal`); exact decimals, arithmetic returns a `result`. |
| [`string`]({{< relref "components/string.md" >}}) | `github.com/go-composites/string/src` | A boxed string value with `Set`/`ToGoString`/`Split`. |
| [`buffer`]({{< relref "components/buffer.md" >}}) | `github.com/go-composites/buffer/src` | A mutable text buffer (StringBuilder); `Append`/`AppendRune`/`Reset` mutate in place and chain — the mutable counterpart to `string`. |
| [`symbol`]({{< relref "components/symbol.md" >}}) | `github.com/go-composites/symbol/src` | An interned, immutable identifier (Ruby's `:name`); `New("x") == New("x")`. |
| [`time`]({{< relref "components/time.md" >}}) | `github.com/go-composites/time/src` | A deterministic instant (no `Now()`) with embedded-tzdata IANA zone support (`In`/`Zone`/`UTC`) plus a `Duration` sub-package; fallible `Parse`/`Add`/`In` return a `result`. |
| [`date`]({{< relref "components/date.md" >}}) | `github.com/go-composites/date/src` | A deterministic calendar date (year/month/day, no time-of-day; no `Today()`), the pure-calendar complement to `time`; fallible `FromYMD`/`Parse`/`AddDays` return a `result`. |
| [`array`]({{< relref "components/array.md" >}}) | `github.com/go-composites/array/src` | Interface-first slice with `Push`/`Pop`/`First`/`Fetch`/`Each` **plus combinators** `Map`/`Filter`/`Reduce`/`Find`/`Any`/`All`; every method returns a `result`. |
| [`dictionary`]({{< relref "components/dictionary.md" >}}) | `github.com/go-composites/dictionary/src` | An interface-first key/value map; `Get` of a missing key returns a `result` error, not a panic. |
| [`set`]({{< relref "components/set.md" >}}) | `github.com/go-composites/set/src` | An unordered collection of unique items with `Union`/`Intersection`/`Difference`. |
| [`orderedset`]({{< relref "components/orderedset.md" >}}) | `github.com/go-composites/orderedset/src` | The insertion-ordered sibling of `set`: same API, but iteration is in first-insertion order. |
| [`sortedset`]({{< relref "components/sortedset.md" >}}) | `github.com/go-composites/sortedset/src` | The comparator-sorted sibling of `set`: items kept in ascending `less`-comparator order (TreeSet-like). |
| [`bag`]({{< relref "components/bag.md" >}}) | `github.com/go-composites/bag/src` | A counted collection (multiset): items carry a multiplicity, with `Count`/`Len`/`DistinctLen` and multiset `Sum`/`Union`/`Intersection`/`Difference`. |
| [`range`]({{< relref "components/range.md" >}}) | `github.com/go-composites/range/src` | An integer interval (inclusive/exclusive end, non-zero step); a zero step is a `result` error, not a panic. |
| [`pair`]({{< relref "components/pair.md" >}}) | `github.com/go-composites/pair/src` | A fixed two-element heterogeneous grouping (`First`/`Second`/`ToArray`). |
| [`proc`]({{< relref "components/proc.md" >}}) | `github.com/go-composites/proc/src` | A first-class callable; `Then` chains steps railway-style, short-circuiting on an error `result`. |
| [`enumerator`]({{< relref "components/enumerator.md" >}}) | `github.com/go-composites/enumerator/src` | A **lazy** sequence (Ruby `Enumerator::Lazy`): lazy `Map`/`Filter`/`Take` over a finite or infinite source; `result`-based terminals. |
| [`compose`]({{< relref "components/compose.md" >}}) | `github.com/go-composites/compose/src` | `Pipe`/`Run` compose `result`-returning steps into one left-to-right, short-circuiting pipeline. |
| [`composites`]({{< relref "components/composites.md" >}}) | `github.com/go-composites/composites` | **Meta-package**: one import re-exporting the whole vocabulary as type aliases + constructors. |
| [`typed`]({{< relref "components/typed.md" >}}) | `github.com/go-composites/typed/src/...` | A **generics** parallel track: `Result[T]`, `Optional[T]`, `Slice[T]` with compile-time type safety. |

### Analyzers {#analyzers}

| Tool | What it enforces |
|------|------------------|
| [`nonnil`]({{< relref "analyzers/nonnil.md" >}}) | A `go vet`-style analyzer that fails the build when an interface carrying `IsNull()` is returned, assigned, or stored as a bare `nil` — the Null-Object invariant. |
| [`respondto`]({{< relref "analyzers/respondto.md" >}}) | A `go vet`-style analyzer that flags `RespondTo` reflective sends to a method that does not exist. |

## Dependency shape {#dependency-shape}

```
compose ─┐
string ──┼▶ array ──▶ result ──▶ error
number ──┤              └─────▶ null
boolean ─┘
dictionary ─▶ result
```

`error` and `null` are leaf packages with no intra-org dependencies; `result`
composes them; the value and collection composites build on `result`. The
[`composites`]({{< relref "components/composites.md" >}}) meta-package imports them all and
re-exports them; [`typed`]({{< relref "components/typed.md" >}}) is standalone with zero
dependencies.

## Quality bar {#quality-bar}

Every library (`src`) package is held to **100% statement coverage** (an org
hard rule, gated in CI) and is built and tested across a native + emulated
**six-architecture** matrix (amd64, arm64, riscv64, loong64, ppc64le, s390x).
Two static analyzers enforce the org's invariants on every repo:
[`nonnil`]({{< relref "analyzers/nonnil.md" >}}) for the Null-Object rule and
[`respondto`]({{< relref "analyzers/respondto.md" >}}) for reflective dispatch. Everything is pure
Go (`CGO_ENABLED=0`) and **BSD-3-Clause**.
