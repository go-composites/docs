# Components

`go-composites` is a set of small, interface-first packages. Each is its own
repository and Go module (under the `github.com/go-composites/` module path; see
the [note on the homepage](../index.md#module-paths)).

## Core

<div class="cop-grid" markdown>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[null](null.md)** — the minimal null-object sentinel.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[error](error.md)** — the error interface and its sentinels.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[result](result.md)** — payload + error wrapper with functional options.
</div>

</div>

## Values & collections

<div class="cop-grid" markdown>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[boolean](boolean.md)** — a boxed boolean with a structural `IsTrue()`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[number](number.md)** — a boxed int/float; arithmetic returns a `result`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[string](string.md)** — a boxed string value.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[symbol](symbol.md)** — an interned, immutable identifier.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[time](time.md)** — a deterministic instant plus a `Duration` span.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[array](array.md)** — interface-first slice with `result`-returning methods and combinators.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[dictionary](dictionary.md)** — an interface-first key/value map.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[set](set.md)** — an unordered collection of unique items.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[range](range.md)** — an integer interval with `result`-returning construction.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[pair](pair.md)** — a fixed two-element grouping.
</div>

</div>

## Composition, meta & generics

<div class="cop-grid" markdown>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[proc](proc.md)** — a first-class callable with railway `Then` composition.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[enumerator](enumerator.md)** — a lazy sequence with `Map`/`Filter`/`Take`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[compose](compose.md)** — `Pipe`/`Run` of `result`-returning steps.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[composites](composites.md)** — the one-import meta-package.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[typed](typed.md)** — the `Result[T]`/`Optional[T]`/`Slice[T]` generics track.
</div>

</div>

The org also ships two static analyzers that enforce its invariants on every
repo — [`nonnil`](../analyzers/nonnil.md) (the Null-Object rule) and
[`respondto`](../analyzers/respondto.md) (reflective dispatch).

## Placeholder repositories

A couple of repositories — `object` and [`is`](is.md) — are currently **empty
placeholders** (a `README` stub, a `LICENSE`, and a `.gitignore`, with no Go
source). They are not yet documented in depth; this site will cover them once
they ship code.
