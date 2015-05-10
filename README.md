Elixir tl;dr
============

This is a short list of Elixir basics made based on the great official Elixir
introduction found at http://elixir-lang.org/getting-started/introduction.html

- Dynamically typed, immutable data types, loops with recursion
- Runs on Erlang VM, can call Erlang code with no overhead
- Parallelism by message passing and green threads ("processes")
- `iex` for REPL, `elixir <file>` to evaluate script, `elixirc` is the
  compiler, `mix` is the build tool
- File naming convention: `.ex` for compiled and `.exs` for script files
- `#` for comment
- `IO.puts "print screen"`, `IO.inspect(data_to_dump_on_screen)`,
  `inspect(data_to_string)`
- Parenthesis are optional in unambiguous situations
- `Foo.bar/2` means function named "bar" of module "Foo" with arity 2
- In REPL `h div` prints documentation for `div`, `h Kernel` for `Kernel`
  module
- `?` suffix convention for function names returning boolean, e.g. `even?/1`
  instead of `is_even/1`
- No line termination. `;` allows joining lines: `x = 1; y = x + 1; y == 2`
- Strings UTF-8
  - `"` double quote is for strings
  - `'` single quote is char list
  - `<<0, 1, 2>>` is byte sequence
  - `~s({"json": "string"})` for "triple-quote" string, i.e. no `"` escaping
- Data structures
  - `[]` for _linked list_ (prepending is fast `[1] ++ []`)
  - `{}` for tuple
  - `:foo` for atom/symbol,
  - `[a: 1, b: 2]` for keyword list (equal to`[{:a, 1}, {:b 2}]`), keys must
    be atoms, ordered, can have same key twice
  - `%{:a => 1, :b => 2}` for traditional map, any value as key, no ordering,
    no duplicate keys (last assignment overwrites)
    - with atom-only keys, `%{a: 1, b: 2}` shorthand can be used
    - pattern-match-able: `%{:k => v} = %{k: 1, k2: 2}; v == 1`
  - Maps and keyword lists implement `Dict` interface
  - Structs are extended maps with default values getting the name of their
    module

    ```elixir
    defmodule Struct do
      defstruct foo: "string", bar: true
    end
    %Struct{} == %Struct{foo: "string", bar: true}
    %Struct{foo: "baz"} == %Struct{foo: "baz", bar: true}
    ```

    with compile-time checks `%Struct{baz: true} # CompileError, unknown key`
    and can be pattern matched like maps `%Struct{foo: var} = %Struct{}; var
    == "string"`
    - Cloning struct with different defaults (and no new keys) using
      structural sharing, use `|` pipe: `lower = %Struct{}; upper =
      %Struct{lower | foo: "STRING"}`
- `a..b` for ranges, `Enum.map(1..3, fn x -> x * 2 end) == [2, 4, 6]`,
  enumerable is a protocol
- `add = fn a, b -> a + b end` anonymous function, called `add.(1,2)`
- convention `*size` named functions constant time, `*length` require
  iteration
- `"interpolate #{variable}"`, `"concat" <> "string"`
- FYI, comparing different data types OK. Valid: `1 < :atom`. Precedence from
  docs.
- Pattern matching
  - `_` discard variable (not readable)
  - `{i, s, a} = {1, "string", :atom}`
  - `{:ok, result} = {:ok, 13}` -> `result == 13`
  - `{:ok, result} = {:error, 13}` -> exception
  - `[head | tail] = [1, 2, 3]` -> `head == 1 and tail == [2, 3]`
- Pin `^` does in-place assert: `x = 1; {x, ^x} = {2, 1}; x == 2`
- Functions' last statement is the return value

  ```elixir
  def name(a, b, c)
    "returned string"
  end
  ```

- Modules group functions

  ```elixir
  defmodule Math do
    def sum(a, b) do
      a + b
    end
  end
  Math.sum(1, 2)
  # > 3
  ```

  creates `Math.sum/2`
- Erlang code can be accessed by their atomic names

  ```elixir
  :math.pow(2, 2)
  # > 4.0
  ```

- `def/2` for public (visible to other modules), `defp/2` for private
  functions
- `captured_fn = &Math.sum/2; captured_fn.(1, 2) == 3`
- `&(&1 + 1)`, `&n` is nth, argument, statement equals `fn x -> x + 1 end`
  - i.e. `&List.flatten(&1, &2)` equals `fn(list, tail) -> List.flatten(list, tail) end`
- Default arguments, can also be statements (evaluated on call)

  ```elixir
  def fun(x \\ IO.puts "hello world") do
    x
  end
  ```

- A parallel "process" is started with `spawn`, each has `pid` for addressing
  the process. Current `pid` read with `self()`.
  - Message are sent to `pid` with `send`, e.g. `send self(), {:some "data}`
  - `receive` blocks and waits for pattern matched messages

  ```elixir
  pid = spawn fn ->
    receive do
      {:msg_a, data} -> "Got tuple matching :msg_a with #{data}"
      "foo" -> "Got string \"foo\""
      some_var -> "Caught something else: #{some_var}"
    after
      1000000 -> "timeout, didn't get anything in 1000ms"
    end
  end
  ```

  - `pid` can be named with `Process.register(pid, :name_for_pid)`

- Guards
  - In `case`

    ```elixir
    case {1, 2} do
      {1, x} when x < 3 -> "guard matched #{x}"
      {1, x} -> "guard passed #{x}"
      _ -> "match rest"
    end
    ```

  - In function definition used for dispatching

    ```elixir
    defmodule Guard do
      def func(a) when a > 0 do "#{a} less than 0" end
      def func(a) when a < 0 do "#{a} greater than 0" end
    end
    Guard.func(1)
    # > "1 less than 0"
    Guard.func(-1)
    # > "-1 greater than 0"
    Guard.func(0)
    # > ** (FunctionClauseError) no function clause matching in Guard.func/1
    ```

- `cond` to do if then else, if none match, error raised

  ```elixir
  cond do
    1 + 1 == 3 -> "if"
    1 + 1 == 4 -> "else if"
    true -> "else"
  end
  ```

- `do/end`: `if true do: (expr, expr, ...)` same as `if true do expr; expr; ... end`
- `Stream` for lazy enumeration, `Enum` for eager
- `|>` for pipe: `[1,2,3] |> Enum.sum == 6`, equal to `Enum.sum([1,2,3]) == 6`
- Protocol (interface in some other languages) defines prototype for
  implementation

  ```elixir
  defprotocol Validation do
    @fallback_to_any true
    def valid?(data)
  end

  defimpl Validation, for: Integer do
    def valid?(i), do: i > 0
  end

  Validation.valid?(-3) == false
  ```

  Fallback for unexpected data types with `@fallback_to_any true` annotation
  before `defprotocol` prototype allows.

  ```elixir
  defimpl Validation, for: Any, do: (def valid?(i), do: false)
  ```

- `for` comprehension is syntactic sugar for enumerations

  ```elixir
  result = for x <- [1,2,3], # "generator"
    y = x + 1,               # temporary variables can be used
    y > 2,                   # "filter"
    y < 4,                   # another filter
  do: y + 1                  # "collectable"
  result == [4]
  ```

  Generator is not same as e.g. Python generator (it can still be enumerable
  lazy stream though). It's just the part producing values.
- `~` for sigil, allowing language extensions, defined with function prototype
  `sigil_{identifier}`. Predefined sigils include e.g. regexps (`sigil_r`)
  `"foo" =~ ~r/foo|bar/`
- Status is usually returned as atom in tuple...

  ```elixir
  File.read("hello") == {:error, :enoent}
  ```

  ...instead of raising (throwing) an error (exception).

  ```elixir
  try do
     raise "fail"
  rescue
     e in RuntimeError -> e
  end
  ```

  Use `defexception` to create own exception types. *Errors are meant for
  unexpected situations, not for control flow*. Process supervision is the
  general way to protect against worst case situations.
- `try`/`catch` can be used to return values from bad API / nested structure
  / function callback loop, should not be used unless it's the only feasible
  option.
- `exit` signals supervisor
