# compose — railway-oriented pipelines

`github.com/go-composites/compose` (Go package `Compose`, imported from the
`src/` sub-directory) is the org's **composition** primitive — the operation
that names go-composites. It wires small `Result`-returning **steps** into a
single pipeline that threads a payload forward along a *success track* and
short-circuits onto an *error track* the instant a step fails.

## API

```go
import Compose "github.com/go-composites/compose/src"

type Step func(input interface{}) Result.Interface

func Pipe(steps ...Step) Step
func Run(input interface{}, steps ...Step) Result.Interface

func Then(transform func(input interface{}) interface{}) Step
func Map(transform func(input interface{}) interface{}) Step
func Recover(step Step, handler func(failed Result.Interface) Result.Interface) Step
func Fail(message string) Step
```

A `Step` receives the payload produced by the previous stage and returns a
[`Result`](result.md). Per the org's null-object discipline, a step never
returns a bare `nil`: it returns either a payload-bearing `Result` (success) or
a `Result` whose [`HasError()`](result.md#haserror) is `true` (failure).

## Behaviour

| Function | Behaviour |
|----------|-----------|
| `Pipe(steps...)` | Composes the steps **left-to-right** into one reusable `Step`. |
| `Run(input, steps...)` | Convenience for `Pipe(steps...)(input)` — compose and run in one call. |
| `Then(transform)` | Lifts an infallible `func(interface{}) interface{}` onto the success track; its return value becomes a successful `Result`'s payload. |
| `Map(transform)` | Alias of `Then`. |
| `Recover(step, handler)` | Runs `handler` only when `step` short-circuits, giving it a chance to return a fallback `Result` and rejoin the success track. On success the `Result` passes through untouched. |
| `Fail(message)` | A `Step` that always short-circuits with an error carrying `message`. |

### Data flow and short-circuiting

`Pipe` wraps its `input` in a successful `Result`, then runs each step in order
on the **previous step's `Payload()`** — data flows forward, left to right. As
soon as a step returns a `Result` with `HasError() == true`, the pipeline
returns that error `Result` **unchanged** and no later step runs. Otherwise the
final step's `Result` is returned.

`Pipe()` with no steps is the **identity track**: it simply wraps its input in a
successful `Result`. The returned `Step` is reusable and safe to run on many
inputs.

## Usage

```go
import (
	Compose "github.com/go-composites/compose/src"
	Error   "github.com/go-composites/error/src"
	Result  "github.com/go-composites/result/src"
)

func parse(input interface{}) Result.Interface {
	n, err := strconv.Atoi(input.(string))
	if err != nil {
		return Result.New(Result.WithError(Error.New("not an integer")))
	}
	return Result.New(Result.WithPayload(n))
}

func validate(input interface{}) Result.Interface {
	if input.(int) <= 0 {
		return Result.New(Result.WithError(Error.New("must be positive")))
	}
	return Result.New(Result.WithPayload(input))
}

// A reusable pipeline: parse -> validate -> double.
pipeline := Compose.Pipe(
	parse,
	validate,
	Compose.Then(func(input interface{}) interface{} { return input.(int) * 2 }),
)

ok := pipeline("21")            // HasError() == false, Payload() == 42
bad := pipeline("not-a-number") // HasError() == true,  Error().Message() == "not an integer"
                                // (validate and the doubling step never run)
```

`Run("21", parse, validate, ...)` is the same as `pipeline("21")` without first
binding the pipeline to a variable.

## Dependencies

`compose` depends on [`result`](result.md) and [`error`](error.md) (and
transitively on [`null`](null.md)).
