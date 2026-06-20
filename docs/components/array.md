# array — interface-first slice

`github.com/go-composites/array` (Go package `Array`, imported from the `src/`
sub-directory) is an **interface-first** wrapper around a `[]interface{}`. Every
method returns a [`result`](result.md), so it composes with the rest of the org.

## API

```go
import Array "github.com/go-composites/array/src"

type Interface interface {
    Each(fn func(int, interface{}) Result.Interface) Result.Interface
    Push(interface{}) Result.Interface
    Pop() Result.Interface
    First() Result.Interface
    Fetch(int) Result.Interface
    Last() Result.Interface
    Clear() Result.Interface
    Copy() Result.Interface
}

func New() Interface
func Each(items []interface{}, fn func(int, interface{}) Result.Interface) Result.Interface
```

| Method | Behaviour |
|--------|-----------|
| `Push(x)` | Appends `x`; returns a result whose payload is the array itself. |
| `Pop()` | Removes and returns the **last** element as the result payload. Panics if the array is empty. |
| `First()` | Returns element `0` as the payload. Panics if empty. |
| `Fetch(i)` | Returns element `i` as the payload. Panics if `i` is out of range. |
| `Last()` | **Buggy — see below.** |
| `Clear()` | Resets to an empty slice; returns an empty result. |
| `Copy()` | Returns an empty result — **not implemented** (no copy is performed). |
| `Each(fn)` | Iterates, calling `fn(index, item)`; short-circuits — see below. |

There is also a package-level `Each(items, fn)` for iterating a plain
`[]interface{}` without constructing an `Array`.

## `Last()` — known bug

!!! danger "`Last()` panics: off-by-one index"
    `Last()` indexes the slice at `len(d.value)` instead of `len(d.value)-1`:

    ```go
    func (d data) Last() Result.Interface {
        return Result.New(
            Result.WithPayload(d.value[len(d.value)]), // out of range
        )
    }
    ```

    `d.value[len(d.value)]` is always one past the last valid index, so `Last()`
    **always panics** with `index out of range [n] with length n` for any
    non-empty array (and on an empty array as well). It never returns. Use
    `Fetch(len-1)` as a workaround until this is fixed. This page documents the
    method as it actually behaves.

## `Each` and the inverted `result`

`Each` stops iterating as soon as the callback's result reports
`HasError() == true`:

```go
for index, item := range d.value {
    if result := fn(index, item); result.HasError() {
        return result
    }
}
return Result.New()
```

Because [`result.HasError()` is inverted](result.md#haserror-inverted-semantics)
in the published modules, the short-circuit fires on a result that has **no**
error and continues on a result that **does** carry one. The package's own tests
encode this: returning a bare `Result.New()` from the callback stops iteration
after the first item, while returning `Result.New(WithError(Error.New(...)))`
lets it run to completion. Keep this inversion in mind when writing callbacks.

## Dependencies

`array` depends on [`result`](result.md) (and transitively on
[`error`](error.md) and [`null`](null.md)).
