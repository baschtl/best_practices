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

### Different implementations on the same function

```
handle_open = fn
  {:ok, file}  -> "Read data: #{IO.read(file, :line)}"    
  {_,   error} -> "Error: #{:file.format_error(error)}"
end

# call it

handle_open.(File.open("some_path_to_a_file"))
```

- depending on the result of `File.open()` the function `handle_open` reads the first line of the file or outputs an error message
- `:file` refers to the Erlang `File` module that can be called through Elixir

### Nesting functions

```
fun1 = fn ->
          fn ->
             "Called!"
          end
       end

# call it

fun1.().()
```

### Remembering of the original environment (closures)

```
fun1 = fn name ->
          fn ->
             "Called #{name}!"
          end
       end

# call it

caller = fun1.("Sebastian")
caller.()
"Called Sebastian!"
```

- the scope defined by the outer function is inherited by the inner function it encloses
- this also works when the inner function itself defines a parameter:

```
fun1 = fn name ->
          fn by ->
             "Called #{name} by #{by}!"
          end
       end

# call it

caller = fun1.("Sebastian")
caller.("Mark")
"Called Sebastian by Mark!"
```

### Functions as arguments

```
times_3 = fn n -> n * 3 end
apply = fn (fun, value) -> fun.(value) end

apply.(times_3, 5)
```

### & notation

`&()` is a shortcut to define an anonymous function.

```
times_3 = &(&1 * 3)
apply = &(&1.(&2))

apply.(times_3, 5)
```

- elixir recognizes calls to a named function and optimizes the anonymous function away if the arity matches:

```
hello = &(IO.puts(&1))
&IO.puts/1
```

- if used with an underlying Erlang function elixir directly maps onto the Erlang function:

```
&abs(&1)
&:erlang.abs/1
```

- lists and tuples can also be turned into functions as `[]` and `{}` are operators:

```
divrem = &{ div(&1, &2), rem(&1, &2) }
divrem.(5, 10)
```

- `&` can also be used to wrap existing functions when giving it its arity:

```
d = &div/2
d.(5, 10)
```
