# Components

`golang-cop` is five small, interface-first packages. Each is its own
repository and Go module (under the `github.com/golang-oop/` module path; see
the [note on the homepage](../index.md#module-paths-vs-repository-org)).

<div class="cop-grid" markdown>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[array](array.md)** — interface-first slice with `result`-returning methods.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[null](null.md)** — the minimal null sentinel.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[result](result.md)** — payload + error wrapper with functional options.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[string](string.md)** — a boxed string value.
</div>

<div class="cop-card" markdown>
<img src="../../assets/logo.svg" alt="">
**[error](error.md)** — the error interface and its sentinels.
</div>

</div>

## Placeholder repositories

The organisation also contains three repositories — `object`, `is`, and
`boolean` — that are currently **empty placeholders** (a `README` stub, a
`LICENSE`, and a `.gitignore`, with no Go source). They are intentionally not
documented here; this site will cover them once they ship code.
