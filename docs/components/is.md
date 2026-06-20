# is — predicate helpers (early / minimal)

`github.com/go-composites/is` is reserved as the org's home for **predicate
helpers** — small functions for asking yes/no questions about composites (for
example, whether a value is null). Following the org convention, the importable
package will live under the `src/` sub-directory
(`github.com/go-composites/is/src`).

!!! warning "Not yet implemented"
    At the time of writing the `is` repository contains only its `LICENSE` and a
    README stub — there is **no `src/` directory and no exported Go code yet**.
    This page is a placeholder and will be filled in once the package ships its
    first predicates. Nothing here should be imported until then.

## API

_None yet._ The package exposes no exported symbols at this time.

## Dependencies

None published yet. As a predicate helper over the org's building blocks, `is`
is expected to depend on [`null`](null.md) and [`result`](result.md) once it
lands.
