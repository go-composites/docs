# Components

`go-composites` is a set of small, interface-first packages. Each is its own
repository and Go module (under the `github.com/go-composites/` module path; see
the [note on the homepage](../index.md#module-paths)).

## Core

<div class="cop-grid" markdown>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[base](base.md)** — the reflective root every composite embeds (`Kind`/`RespondTo`/`Methods`).
</div>

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
**[bignumber](bignumber.md)** — an arbitrary-precision integer over `math/big.Int`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[rational](rational.md)** — an exact rational number over `math/big.Rat`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[complex](complex.md)** — a complex number `a+bi` over `complex128`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[bigfloat](bigfloat.md)** — an arbitrary-precision float over `math/big.Float`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[string](string.md)** — a boxed string value.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[buffer](buffer.md)** — a mutable text buffer (StringBuilder), the counterpart to `string`.
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
**[date](date.md)** — a deterministic calendar date, the pure-calendar complement to `time`.
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
**[orderedset](orderedset.md)** — the insertion-ordered sibling of `set`.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[sortedset](sortedset.md)** — the comparator-sorted sibling of `set`.
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

One repository — [`is`](is.md) — is currently an **empty placeholder** (a
`README` stub and a `LICENSE`, with no Go source). It is not yet documented in
depth; this site will cover it once it ships code.
