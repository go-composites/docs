# go-composites

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
by inheritance. Fallible methods return a [`result`](components/result.md) value
rather than the Go idiom of multiple return values, so the packages compose with
one another, and the **Null-Object** pattern runs throughout — a fresh `Result`,
`Error` or `Null` is a real, non-nil value, so callers branch on `HasError()`
instead of nil checks.

## Module paths

!!! note "Org and module path"
    These projects live in the GitHub organisation **`go-composites`**, and their
    Go module paths match it — **`github.com/go-composites/<name>`**. For example
    the `array` repository declares `module github.com/go-composites/array`. The
    importable package lives under the `src/` sub-directory (e.g.
    `github.com/go-composites/array/src`, package `Array`).

    The one exception is the [`composites`](components/composites.md)
    **meta-package**, which is imported at the module root —
    `import "github.com/go-composites/composites"`.

## Components

| Module | Import path | What it does |
|--------|-------------|--------------|
| [`null`](components/null.md)   | `github.com/go-composites/null/src` | A minimal null-object sentinel used as the default `result` payload. |
| [`error`](components/error.md) | `github.com/go-composites/error/src` | The `error` interface (`Message`/`IsNull`) plus null and `method-not-implemented` sentinels. |
| [`result`](components/result.md) | `github.com/go-composites/result/src` | Wraps a `payload` plus an `error` built with functional options; `HasError()` is `true` only on a real error. |
| [`boolean`](components/boolean.md) | `github.com/go-composites/boolean/src` | A boxed boolean (`IsTrue`/`IsFalse`/`Equal`); its structural `IsTrue()` is what `array`'s combinators treat as truthy. |
| [`number`](components/number.md) | `github.com/go-composites/number/src` | A boxed numeric value (int/float); arithmetic returns a `result`, so division by zero is an error value, not a panic. |
| [`string`](components/string.md) | `github.com/go-composites/string/src` | A boxed string value with `Set`/`ToGoString`/`Split`. |
| [`symbol`](components/symbol.md) | `github.com/go-composites/symbol/src` | An interned, immutable identifier (Ruby's `:name`); `New("x") == New("x")`. |
| [`time`](components/time.md) | `github.com/go-composites/time/src` | A deterministic instant (no `Now()`) plus a `Duration` sub-package; fallible `Parse`/`Add` return a `result`. |
| [`array`](components/array.md) | `github.com/go-composites/array/src` | Interface-first slice with `Push`/`Pop`/`First`/`Fetch`/`Each` **plus combinators** `Map`/`Filter`/`Reduce`/`Find`/`Any`/`All`; every method returns a `result`. |
| [`dictionary`](components/dictionary.md) | `github.com/go-composites/dictionary/src` | An interface-first key/value map; `Get` of a missing key returns a `result` error, not a panic. |
| [`set`](components/set.md) | `github.com/go-composites/set/src` | An unordered collection of unique items with `Union`/`Intersection`/`Difference`. |
| [`range`](components/range.md) | `github.com/go-composites/range/src` | An integer interval (inclusive/exclusive end, non-zero step); a zero step is a `result` error, not a panic. |
| [`pair`](components/pair.md) | `github.com/go-composites/pair/src` | A fixed two-element heterogeneous grouping (`First`/`Second`/`ToArray`). |
| [`proc`](components/proc.md) | `github.com/go-composites/proc/src` | A first-class callable; `Then` chains steps railway-style, short-circuiting on an error `result`. |
| [`compose`](components/compose.md) | `github.com/go-composites/compose/src` | `Pipe`/`Run` compose `result`-returning steps into one left-to-right, short-circuiting pipeline. |
| [`composites`](components/composites.md) | `github.com/go-composites/composites` | **Meta-package**: one import re-exporting the whole vocabulary as type aliases + constructors. |
| [`typed`](components/typed.md) | `github.com/go-composites/typed/src/...` | A **generics** parallel track: `Result[T]`, `Optional[T]`, `Slice[T]` with compile-time type safety. |

### Analyzers

| Tool | What it enforces |
|------|------------------|
| [`nonnil`](analyzers/nonnil.md) | A `go vet`-style analyzer that fails the build when an interface carrying `IsNull()` is returned, assigned, or stored as a bare `nil` — the Null-Object invariant. |
| [`respondto`](analyzers/respondto.md) | A `go vet`-style analyzer that flags `RespondTo` reflective sends to a method that does not exist. |

## Dependency shape

```
compose ─┐
string ──┼▶ array ──▶ result ──▶ error
number ──┤              └─────▶ null
boolean ─┘
dictionary ─▶ result
```

`error` and `null` are leaf packages with no intra-org dependencies; `result`
composes them; the value and collection composites build on `result`. The
[`composites`](components/composites.md) meta-package imports them all and
re-exports them; [`typed`](components/typed.md) is standalone with zero
dependencies.

## Quality bar

Every library (`src`) package is held to **100% statement coverage** (an org
hard rule, gated in CI) and is built and tested across a native + emulated
**six-architecture** matrix (amd64, arm64, riscv64, loong64, ppc64le, s390x).
Two static analyzers enforce the org's invariants on every repo:
[`nonnil`](analyzers/nonnil.md) for the Null-Object rule and
[`respondto`](analyzers/respondto.md) for reflective dispatch. Everything is pure
Go (`CGO_ENABLED=0`) and **BSD-3-Clause**.
