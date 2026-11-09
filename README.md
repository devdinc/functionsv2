# Functions v2

Create Java functional-interface instances from concise Skript lambda expressions or indented lambda sections. Supports zero, one, or two arguments and provides helpers for invoking the resulting objects.

## Requirements

- Skript 2.15+
- skript-reflect 2.6+

## Installation

```
package install https://github.com/devdinc/functionsv2
```

## Lambda expressions

Use `lambda` followed by an optional argument list and a one-line body:

```skript
set {_double} to lambda {_value}: {_value} * 2
set {_result} to run lambda {_double} with 5

set {_supplier} to supplier: 42
set {_value} to run lambda {_supplier}

set {_printer} to consumer {_message}: broadcast {_message}
run lambda {_printer} with "Hello"
```

For `lambda`, the body is interpreted automatically: an effect body creates a no-return lambda, while an expression body returns a value. Prefix the body with `-` to force no return, or `+` to force a return:

```skript
set {_action} to lambda {_value}: - broadcast {_value}
set {_mapper} to lambda {_value}: + {_value} parsed as number
```

## Functional interface expressions

The package also provides explicit constructors:

| Constructor | Java interface | Arguments |
|---|---|---:|
| `supplier: <body>` / `getter: <body>` | Supplier | 0 |
| `runnable: <body>` / `runner: <body>` | Runnable | 0 |
| `consumer %objects%: <body>` / `accepter` | Consumer | 1 |
| `function %objects%: <body>` / `applier` | Function | 1 |
| `biconsumer %objects%: <body>` / `biaccepter` | BiConsumer | 2 |
| `bifunction %objects%: <body>` / `biapplier` | BiFunction | 2 |

For example:

```skript
set {_sum} to bifunction {_a}, {_b}: {_a} + {_b}
set {_answer} to run lambda {_sum} with 3 and 4
```

## Multi-line lambda sections

Use an indented block when the lambda needs multiple statements:

```skript
condition set {_callback} to lambda {_value}:
    if {_value} is set:
        broadcast "Received %{_value}%"
    else:
        broadcast "No value"
```

The section form can be prefixed with `-` to force no return or `+` to force a return. The condition assigns the generated function object to the target expression.

## Invocation

```skript
set {_result} to run lambda {_function} with {_argument}
run lambda {_callback} with {_argument}
```

`run lambda` is available as an expression (to retrieve a return value) and as an effect (to invoke without using a return value). Lambdas accept up to two arguments.

## Dependencies

This package declares dependencies on [single-line-section](https://github.com/devdinc/single-line-section) and [pacskage-reorder](https://github.com/devdinc/pacskage-reorder).

## Source and tests

- Implementation: `src/functionsv2.sk`
- Tests: `-tests/`
