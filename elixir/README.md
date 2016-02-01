# Best practices in Elixir

## Table of contents

* [Basics](#basics)
  * [Collection types](#collection-types)
  * [Binaries](#binaries)
  * [Truth](#truth)
  * [Operators](#operators)
* [Anonymous functions](#anonymous-functions)

## Basics

### Collection types

#### Tuples

`{:ok, 1, "Sebastian"}`

- can be used in pattern matching
- usually two to four elements

#### Lists

`[1, 2, 3]`

- linked data structure
- easy to traverse linearly
- expensive to get the nth element
- but cheap to get head and tail

- specific operators:
  - `++` (concatenation)
  - `--` (difference)
  - `in` (membership)

##### Keyword lists

```
[ name: "Sebastian", age: 29]
[ {:name, "Sebastian"}, {:age, 29} ]
```

- if the last argument of a method brackets can be omitted
- use for command-line parameters or passing around options, etc.

#### Maps

```
my_map = %{ name: "Sebastian", age: 29 }

my_map[:name]
> "Sebastian"
my_map.name   # only if key is atom
> "Sebastian"
```

- compared to keyword lists only allows unique keys
- use if an associative array is needed
- efficient if they grow

### Binaries

- basic syntax: `my_binary = << 1, 2 >>`, packs successive intergers into bytes
- with additional information: `my_binary = << 1 :: size(2), 5 :: size(4), 3 :: size(2) >>`, each field has separate size

### Truth

- `true`, `false`, `nil` (all aliases for their respective atoms, i.e., `:true`, `:false`, `:nil`)
- `nil` is considered to be false in boolean contexts
- everything else than `false` and `nil` is truthy

### Operators

#### Comparison

- `a === b`, strict equality (`1 === 1.0` is false)
- `a !== b`, strict inequality (`1 !== 1.0` is true)
- `a == b`, value equality (`1 == 1.0` is true)
- `a != b`, value inequality (`1 != 1.0` is false)

#### Boolean

- `or`, `and`, `not` expect `true` or `false` as their first operator
- `||`, `&&`, `!`, any value apart from `false` and `nil` is treated as `true`

#### Arithmetic

- `+ - * / div rem`
- `/` returns floating point number, `div` returns integer

## Anonymous functions

### Example

```
times = fn (a, b) -> a * b end
times.(2, 5)

hello = fn -> IO.puts "Hello!" end
hello.()
```
- the dot indicates the function call for an anonymous function
- when arguments are passed to a function Elixir tries to match them to the parameters
