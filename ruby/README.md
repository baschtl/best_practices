# Best practices in Ruby

## Table of contents

* [String](#string)
* [Regular Expressions](#regular-expressions)
* [Struct](#struct)
* [Classes](#classes)
* [Modules](#modules)
* [Blocks](#blocks)
* [Metaprogramming](#metaprogramming)
* [Tools](#tools)
* [Debugging](#debugging)

## String

### In general

- mutual, e.g., `a = b = "hello"; b\[-1\] = a`, this changes `a` and `b` to `hella`
- single-quoted:
  - do not escape quotes like `\\`
  - cannot be used to embed expressions like `#{variable}`
- double-quoted:
  - can include tabs `\\t`, newlines `\\n` and `'`
  - can be used to embed expressions like `#{variable}`
- arbitrary quote mechanism:
  - can be used with a mixture of quote types
- single-quote style:
  - `%q( "Stop", he said, "This ' is enough!" )`
- double-quote style:
  - `%Q( I can handle expressions like #{variable}. )`
- multiline:
  - adding a new line within a string (double-quote or arbitrary quote style) literally adds it to the string
  - this can be avoided: `first line \\ second line`

### Formatting

- a short-hand version for `sprintf` is `%`:
  - `str = 'file_%02d%02d%d' % %w( 3 12 2014 ) # file_03122014`

## Regular Expressions

### In general

- `=~`
  - returns a number (i.e., the index where the first match was found) or
  - nil (i.e., if no match was found)

- `!~`
  - uses `=~` for matching
  - returns true if two objects don't match, otherwise false

### Matching

- `\A` matches at the beginning of a string
- `^` matches at the beginning of a string or at the beginning of each line within a string
- `\z` matches at the end of a string
- `$` matches at the end of a string or at the end of each line within a string

### Equality

[This article](https://mauricio.github.io/2011/05/30/ruby-basics-equality-operators-ruby.html) gives a good overview.

- there is `eql?`, `equal?`, `==` and `===`
- `equal?` (comes from Object):
  - object equality (i.e., objects with the same hash key are equal)
-  `==`:
  - defaults to `equal?`
- `===`:
  - defaults to `==`
  - only used in case statements
- `eql?`:
  - defaults to `equal?`
  - used in combination with hash in Hash to find objects stored in a hash by their key
  - actually used to determine if two keys are the same

`===` in case statements can be used in combination with custom matcher classes to shorten the statement like this:

```ruby
class Success

  def self.===(item)
    item.status >= 200 && item.status < 300
  end

end

class Empty

  def self.===(item)
    item.response_size == 0
  end

end

case http_response
  when Empty
    puts "response was empty"
  when Success
    puts "response was a success"
end
```

### Integers

- classes with a natural ordering like Fixnum and Float implement `<=>`
- `<=>` evaluates to -1 (less than), 0 (equal) or 1 (greater than)
- the Comparable module of Ruby implements `==`, `<`, `<=`, `>=` and `>` based on `<=>`
  - makes implementing a collection of objects easier

### Class

- `===` is an alias of `kind_of?`, used for case statements

## Struct

Ways to create a struct:

```ruby
# Inheritance
class Person < Struct.new(:name, :age)
  def to_s
    "#{name} - #{age}"
  end
end
```
This creates a new entry in the ancestor chain.

```ruby
# Block
Person = Struct.new(:name, :age) do
  def to_s
    "#{name} - #{age}"
  end
end
```
With the block version you cannot create constants within `Person`. It will be a global one and not scoped.

```ruby
# Re-open
Person = Struct.new(:name, :age)

class Person
  def to_s
    "#{name} - #{age}"
  end
end
```
The re-open version circumvents both aforementioned issues.

## Classes

### Singleton methods

- defines a new method for a single instance

```ruby
my_instance = Object.new

def my_instance.new_method
  ":-D"
end

# or

class << my_instance

  def new_method
    ":-D"
  end

end
```

### Singleton Class

- sits between an instance of a class and the class itself
- is created lazily
- holds singleton methods of the instance

```ruby
singleton_class = class << my_instance
  self
end
```

### Class methods

- use the concept of singleton methods
- a class method is only defined for a single instance of Class

```ruby
my_instance = Object

def my_instance.new_method
  ":-D"
end

# Call with: my_instance.new_method or Object.new_method

# or

def Object.new_method
  ":-D"
end

# or

class Object

  class << self

    def new_method
      ":-D"
    end

  end

end
```

Making class methods private

- `private` is a method and only changes the visibility of instance methods
- to make singleton or class methods private use the following:

```ruby
# Singleton method example

foo = Foo.new

class << foo

  def speak
    # implementation_details
  end

  private

  def implementation_details
    puts "hello!!!"
  end

end

# Class method example

class Foo

  class << self

    def speak
      # implementation_details
    end

    private

    def implementation_details
      puts "hello!!!"
    end

  end

end

# or

private_class_method
```

### Class variables

- defined using `@@my_class_variable`
- when setting a class variable Ruby checks if it is already defined in the current class
  - if it is it uses it,
  - if it is not defined in the current class it searches in the inheritance tree:
    - if it finds the variable in one of the super classes it sets it there,
    - if it does not find it, it creates the variable on the current class
- when looking up a value for a class variable Ruby follows the same approach as when setting it
  - if it does not find a variable definition it throws a NameError exception
- Attention: this means that class variables tend to wander around!
  - if the same class variable is defined in a super class and inheriting class the one of the super class is taken
- can also be applied to modules

### Class instance variables
- `@variables` that are attached to an instance of a class
- do not have the same subclass issues as class variables

```ruby
class MyClass

  class << self

    # Auto generate getter and setter for class instance variable
    attr_accessor :variable

  end

  def self.set_my_variable=(variable)
    # set an instance variable of class MyClass
    @variable = variable
  end

end
```

## Modules

```ruby
module MyModule

 def self.module_level_method
   # I am a module level method and can be called with
   # MyModule.module\_level\_method
 end

 def module_level_method
   # I am a method that can be called when mixed into a class
 end

end
```

### Extension

- adds methods of a module as class methods to a class (i.e., includes module to the singleton class)

```ruby
class MyClass

  class << self

    include SomeModule

  end

end

# is equal to

class MyClass

  extend SomeModule

end
```

### Inheritance

- including a module into a class inserts it before the superclass to the inheritance tree

```ruby
class MyClass

  extend SomeModule

end
```

- MyClass Instance <- SomeModule <- MyClass
- the class of an instance will still be the class itself
- however, using `instance.kind_of?(SomeModule)` will return true
- using `instance.ancestors` returns an array that includes SomeModule
- methods that were included can be overridden within the class
- if two modules with the same method are included into a class the method of the module included most recently will be called

## Blocks

- Example for a custom iterator implementation:

```ruby
def each_word

  words_array = words
  index = 0

  while index < words_array.size do
    yield(words_array[index])
    index += 1
  end

end
```

- blocks can throw exceptions and should handle the teardown of resources (i.e., begin...ensure...end), same applies to break and return in blocks

### Execute around

If something has to happen in a certain order:

```ruby
def do_something
  with_logging('load') { @doc = Document.load('filename.txt') }
  # ...
  with_logging('save') { @doc.save }
end

def with_logging(description)
  @logger.debug("Starting #{description}")
  yield
  @logger.debug("Completed \#{description}")
rescue
  @logger.error("\#{description} failed")
  raise
end
```

Only pass in arguments to an execute around if they are used within the
execute around method itself.

When return values of the given block is needed:

```ruby
def with_logging(description)
  @logger.debug("Starting #{description}")
  return_value = yield
  @logger.debug("Completed #{description}")
  return_value
rescue
  @logger.error("#{description} failed")
  raise
end
```

If setting up objects:

```ruby
class SomeClass

  def initialize(name)
    @name = name
    yield(self) if block_given?
  end

end

some = SomeClass.new('John') do |s|
  s.age = 16
end
```

### Explicit blocks

```ruby
def run_that_block(&that_block)
  that_block.call if that_block
end
```

Usage as callbacks:

```ruby
class Document

  def on_load(&block)
    @load_listener = block
  end

  def load
    # Load file
    @load_listener.call if @load_listener
  end

end

doc.on_load do |doc|
  puts "Loaded!"
end
```

### Proc vs. lambda

- lambdas are strict concerning the number of arguments given to it, Procs are not
- a return statement in a Proc returns from the method that created the block, a lambda only returns from the block
- a block drags the local context it was created in with it:

```ruby
def some_method
  arr = Array.new(100000000)
  # Do some thing with arr
  # arr = nil would remove the reference, hence it will not be available anymore in the following block

  doc.on_load do |d|
    # arr is still available in this block and will be hold until the block gets destroyed
    puts "Loaded!"
  end

end
```

- a short-hand version of `proc.call` is `proc.()`

## Metaprogramming

### Hooks

- use `self.inherited`, `self.included`, `at_exit`, `method_missing`, `method_added`, `trace_var` (see the Kernel module)

### const_missing

- gets called when a non-existing constant gets called
- has to be a class method

### method_missing

- is called when a non-existing method gets called

### Delegation with method_missing

```ruby
def method_missing(name, *args)
  referenced_obj.send(name, *args)
end
```

- alternatively, inherit from SimpleDelegator (from delegate.rb) handles the delegation via `method_missing`

### Magic methods

- this mechanism can also be used to provide “magic methods”, methods that don't really exist
- method_missing handles calls to not implemented methods
- as long as the method name complies with the format that is known to `method_missing` some behavior is executed

```ruby
class FormLetter << Document

  def replace_word(old_word, new_word)
    # ...
  end

  def method_missing(name, *args)
    string_name = name.to_s
    return super unless string_name =~ /^replace_\w+/
    old_word = extract_old_word(string_name)
    replace_word(old_word, args.first)
  end

  def extract_old_word(name)
    # ...
  end

end
```

- keep attention to the fact that `respond_to?` that is implemented by Object only knows about existing methods

```ruby
class FormLetter << Document

  # ...

  def respond_to?(name)
    string_name = name.to_s
    return true if string_name =~ /^replace_\w+/
    super
  end

  # ...

end
```

### Open classes

- classes in Ruby can be altered after they were defined, hence they are called open classes
- defining a class in Ruby multiple times in fact means adding additional behavior to the first definition of this class

### Monkey Patching

- defining a method of a class a second time overwrites the first definition of the method

### Renaming methods

- `alias_method` can be used to copy (!) a method definition and to give it a different name
- this comes in handy if you overwrite an existing method but still need its original behavior

### Self-Modifying classes

- the Ruby interpreter reads a class file from top to bottom
- hence, it is possible to create a class that defines methods depending on some condition:

```ruby
def MyClass

  def self.enable_encryption(enable)
    if enable
      def encrypt_string(string)
        # encrypt string
      end
    else
      def encrypt_string(string)
        # do not encrypt string
      end
    end
  end

  enable_encryption(ENCRYPTION_ENABLED)

end
```

- the if-statement on class level is executed exactly once when the class is read
- method definitions using def need a fixed name
- when the name is defined dynamically def is not an option
- in this case you can use `class_eval`:

```ruby
code = %Q(

  def #{method_name}(text)
    # do something
  end

)

class_eval(code)
```

- however, this is not the best option because there is a Ruby API mechanism that does exactly that:

```ruby
define_method(method_name) do |text|
  # do something
end
```

- the block parameters get the method parameters
- examples in Ruby/Rails for the use of dynamically defining methods: `attr_reader`, `attr_writer`, Ruby's forwardable.rb (generates delegating methods, similar to delegate.rb but does not use `method_missing`)

## Internal DSLs

Instead of using:

```ruby
class SomeClass

  def initialize(&block)
    block.call(self) if block
  end

end
```

that is called via:

```ruby
SomeClass.new do |s|
  s.some_method
end
```

a step towards an internal DSL is to use:

```ruby
class SomeClass

 def initialize(&block)
   instance_eval(&block) if block
 end

end
```

which allows:

```ruby
SomeClass.new do
  some_method
end
```

The value of self within the block is set to the created instance of SomeClass.

- to build on that we could also simplify the creation of the SomeClass instance

```ruby
class SomeClass

  def initialize_with_file(path)
    # the second parameter tells instance\_eval where the code it received came from
    instance_eval(File.read(path), path)
  end

  # ...

end

# Somewhere in the loading of the application

some = SomeClass.new

some.initialize_with_file('/path/to/definition.rb')
```

- this approach allows a clean DSL specification without any instantiation of the SomeClass class
- this approach can also be combined with `method_missing` to handle undefined methods

## Tools

### IRB

IRB accepts custom inspectors during start up:

```ruby
$ irb -f --inspect yaml

irb(main)> Person = Struct.new(:name, :age)
=> --- !ruby/class 'Person'

irb(main)> Person.new("Sebastian", 29)
=> --- !ruby/struct:Person
name: Sebastian
age: 29
```

## Debugging

### By hand

- the caller of the this code: `caller`

- the implementation source of a method: `method(:a_method_name).source_location`
- the implicit implementation source of a method:
```ruby
tp = TracePoint.new(:call) do |x|
  puts x
end
tp.enable
# ... code to analyse
tp.disable
```

- if a method calls `super` where does this call go: `method(:a_method_name).super_method.source_location`

- to find out where an exception got raised:
  - run the Ruby script with the `-d` flag: `bundle exec ruby -d script/rails runner my_script.rb`
  - run an arbitrary command: `ruby -d -S rspec`
  - to give every Ruby process the flag: `RUBYOPT=-d rspec`

- to find out when an object is mutated: use `object.freeze` and look at the stack trace of the raised exception
