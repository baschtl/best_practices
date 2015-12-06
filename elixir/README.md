# Best practices in Elixir

## Collection types

### Tuples

`{:ok, 1, "Sebastian"}`

- can be used in pattern matching
- usually two to four elements

### Lists

`[1, 2, 3]`

- linked data structure
- easy to traverse linearly
- expensive to get the nth element
- but cheap to get head and tail

- specific operators:
  - `++` (concatenation)
  - `--` (difference)
  - `in` (membership)

#### Keyword lists

```
[ name: "Sebastian", age: 29]
[ {:name, "Sebastian"}, {:age, 29} ]
```

- if the last argument of a method brackets can be omitted
- use for command-line parameters or passing around options, etc.

### Maps

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
