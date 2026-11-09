# Functions v2

Create Java functional-interface instances from Skript lambda expressions and multiline lambda sections. The resulting objects are Java proxies that can be passed to Java APIs, including Java streams, or stored for later invocation.

## Requirements

- Skript 2.15+
- skript-reflect 2.6+
- [single-line-section](https://github.com/devdinc/single-line-section), loaded before this package
- [pacskage-reorder](https://github.com/devdinc/pacskage-reorder)

Skript loads scripts alphabetically. Ensure dependency scripts load first (for example, prefix their filenames with `!`).

## Installation

```sh
package install https://github.com/devdinc/functionsv2
```

## Single-line lambdas

Use `lambda` with an optional argument list and a one-line body:

```skript
set {_double} to lambda {_value}: {_value} * 2
set {_result} to run lambda {_double} with 5

set {_supplier} to supplier: 42
set {_value} to run lambda {_supplier}

set {_printer} to consumer {_message}: broadcast {_message}
run lambda {_printer} with "Hello"
```

For generic `lambda` expressions, the body is interpreted automatically: effect bodies create non-returning lambdas, while expression bodies return a value. Prefix the body with `-` to force non-returning behavior or `+` to force returning behavior:

```skript
set {_action} to lambda {_value}: - broadcast {_value}
set {_mapper} to lambda {_value}: + {_value} parsed as number
```

## Multiline lambda sections

Use an indented block when the lambda needs multiple statements. Assign it directly with `set`:

```skript
set {_callback} to lambda {_value}:
    if {_value} is set:
        broadcast "Received %{_value}%"
    else:
        broadcast "No value"
```

The multiline generic lambda form is written as a condition that assigns the generated function object to the target expression; do not put the literal word `condition` before `set`.

For a non-returning multiline lambda, prefix the section with `-`. For a returning multiline lambda, prefix it with `+` and use `return` in the body:

```skript
set {_task} to lambda-:
    broadcast "working"
    broadcast "still working"

set {_double} to lambda+ {_value}:
    broadcast "doubling %{_value}%"
    return {_value} * 2
```

## Explicit functional-interface constructors

The package also provides typed aliases that select the Java interface and expected argument count:

| Skript constructor | Java interface | Arguments |
|---|---|---:|
| `supplier:` / `getter:` | `Supplier<T>` | 0 |
| `runnable:` / `runner:` | `Runnable` | 0 |
| `consumer %objects%:` / `accepter` | `Consumer<T>` | 1 |
| `function %objects%:` / `applier` | `Function<T, R>` | 1 |
| `biconsumer %objects%:` / `biaccepter` | `BiConsumer<T, U>` | 2 |
| `bifunction %objects%:` / `biapplier` | `BiFunction<T, U, R>` | 2 |

The aliases can also be used with multiline sections. Use `new` for the multiline constructor form:

```skript
set {_task} to new runnable:
    broadcast "working"
    broadcast "still working"

set {_double} to new function {_value}:
    broadcast "doubling %{_value}%"
    return {_value} * 2
```

Returning multiline lambdas require an explicit `return`. Non-returning multiline lambdas should contain effects and no return statement.

## Java interface mapping

The interface is selected from the lambda's argument count and whether it returns a value:

| Arguments | Returning | Non-returning |
|---:|---|---|
| 0 | `Supplier<T>` | `Runnable` |
| 1 | `Function<T, R>` | `Consumer<T>` |
| 2 | `BiFunction<T, U, R>` | `BiConsumer<T, U>` |

Lambdas support zero, one, or two arguments. More than two parameters are not supported. To pass more data, use a composite object such as a list, map, or custom object.

## Invoking lambdas

Use `run lambda` as an expression when you need the return value, or as an effect when you only need to invoke the lambda:

```skript
set {_result} to run lambda {_function} with {_argument}
run lambda {_callback} with {_argument}
```

You can also invoke the underlying Java interface method directly:

- `Supplier` / `Runnable`: `.get()` / `.run()`
- `Function` / `Consumer`: `.apply(value)` / `.accept(value)`
- `BiFunction` / `BiConsumer`: `.apply(a, b)` / `.accept(a, b)`

## Limitations

- Only zero to two lambda parameters are supported.
- Variable lists as lambda parameters are not supported.
- Array indexing inside lambdas may not work reliably; use `spread(...)` before passing arrays.
- Imported classes or complex expressions may fail in inline lambdas. Prefer a multiline lambda or move the logic into a regular Skript function.

## Source and tests

- Implementation: `src/functionsv2.sk`
- Tests: `-tests/`
