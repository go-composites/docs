# go-composites

A small collection of **pure-Go composites** for **Composition-Oriented
Programming**: interface-first building blocks — an `array`, a `null` sentinel,
a `result` wrapper, a boxed `string`, and an `error` interface with sentinels.
No cgo, no third-party runtime dependencies (only `go-spew` in a demo `main`).

The design is deliberately composition-oriented: each package exposes an
`Interface` and a private `data` struct constructed through a `New(...)`
function (several use functional options such as `WithPayload` / `WithGoString`).
Composites are assembled by composing and delegating to smaller composites, never
by inheritance. Methods return a [`result`](components/result.md) value rather
than the Go idiom of multiple return values, so the packages compose with one
another.

## Module paths

!!! note "Org and module path"
    These projects live in the GitHub organisation **`go-composites`**, and their
    Go module paths match it — **`github.com/go-composites/<name>`**. For example
    the `array` repository declares `module github.com/go-composites/array`. The
    importable package lives under the `src/` sub-directory (e.g.
    `github.com/go-composites/array/src`, package `Array`).

## Components

| Module | Import path | What it does |
|--------|-------------|--------------|
| [`array`](components/array.md) | `github.com/go-composites/array/src` | Interface-first slice of `interface{}` with `Push`/`Pop`/`First`/`Fetch`/`Each`; every method returns a `result`. |
| [`null`](components/null.md)   | `github.com/go-composites/null/src` | A minimal null sentinel value used as the default `result` payload. |
| [`result`](components/result.md) | `github.com/go-composites/result/src` | Wraps a `payload` plus an `error` built with functional options. |
| [`string`](components/string.md) | `github.com/go-composites/string/src` | A boxed string value with `Set`/`ToGoString`/`Split`. |
| [`error`](components/error.md) | `github.com/go-composites/error/src` | The `error` interface (`Message`/`IsNull`) plus null and `method-not-implemented` sentinels. |

## Dependency shape

```
string ──▶ array ──▶ result ──▶ error
                         └─────▶ null
```

`error` and `null` are leaf packages with no intra-org dependencies; `result`
composes them; `array` and `string` build on `result`.

## Status

These are early, small primitives (Go `1.21.6`, module versions are
`v0.0.0` pseudo-versions). A couple of published methods have **known bugs** —
see [`array.Last()`](components/array.md#last-known-bug) and
[`result.HasError()`](components/result.md#haserror-inverted-semantics). They
are documented here as they actually behave, not as they were intended.
