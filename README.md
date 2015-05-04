Elixir tl;dr
============

Shortlist of basics grabbed Elixir introduction at http://elixir-lang.org/getting-started/introduction.html

This document is to get a quick idea of some Elixir basics and code you might
be reading. Check the Elixir site for actual high quality documentation.


 - `.ex` for compiled and `.exs` for script files, run `elixir file`
 - `#` for comment
 - Data types immutable, loops with recursion
 - No line termination. `;` allows joining lines: `x = 1; y = x + 1; y == 2`
 - `Foo.bar/2` means function named "bar" of module "Foo" with arity 2
 - Strings UTF-8
   - `"` double quote is for strings
   - `'` single quote is char list
   - `<<0, 1, 2>>` is byte sequence
 - Data structures
   - `[]` for _linked list_ (prepending is fast `[1] ++ []`)
   - `{}` for tuple
   - `:foo` for atom/symbol,
   - `[a: 1, b: 2]` for keyword list (equal to`[{:a, 1}, {:b 2}]`), keys must
     be atoms, ordered, can have same key twice
   - `%{:a => 1, :b => 2}` for traditional map, any value as key, no ordering,
     no duplicate keys (last assignment overwrites)
     - with atom-only keys, `%{a: 1, b: 2}` shorthand can be used
     - pattern-matchable: `%{:k => v} = %{k: 1, k2: 2}; v == 1`
   - Maps and keyword lists implement `Dict` interface
 - `a..b` for ranges, `Enum.map(1..3, fn x -> x * 2 end) == [2, 4, 6]`,
   enumerable is a protocol
 - `add = fn a, b -> a + b end` anonymous function, called `add.(1,2)`
 - convention `*size` named functions constant time, `*length` require
   iteration
 - `"intepolate #{variable}"`, `"concat" <> "string"`
 - FYI, comparing different datatypes ok. Valid: `1 < :atom`. Precedence from
   docs.
 - Pattern matching
   - `_` discard variable (not readable)
   - `{i, s, a} = {1, "string", :atom}`
   - `{:ok, result} = {:ok, 13}` -> `result == 13`
   - `{:ok, result} = {:error, 13}` -> exception
   - `[head | tail] = [1, 2, 3]` -> `head == 1 and tail == [2, 3]`
 - Pin `^` does in-place assert: `x = 1; {x, ^x} = {2, 1}; x == 2`
 - Guards
   - In `case`
     ```
     case {1, 2} do
       {1, x} when x < 3 -> "guard matched #{x}"
       {1, x} -> "guard passed #{x}"
       _ -> "match rest"
     end
     ```
   - In function deifnition
     ```
     def reduce(x) when x > 10
       x - 1
     end
     def reduce(x)
       x
     end
     ```
 - `cond` to do if then else, if none match, error raised
   ```
   cond do
     1 + 1 == 3 -> "if"
     1 + 1 == 4 -> "else if"
     true -> "else"
   end
   ```
 - `do/end`: `if true do: (expr, expr, ...)` same as `if true do expr; expr; ... end`
 - Modules group functions
   ```
   defmodule Math do
     def sum(a, b) do
       a + b
     end
   end
   ```
   creates `Math.sum/2`
 - `def/2` for public (visible to other modules), `defp/2` for private
   functions
 - `captured_fn = &Math.sum/2; captured_fn.(1, 2) == 3`
 - `&(&1 + 1)`, `&n` is nth, argument, statement equals `fn x -> x + 1 end`
   - i.e. `&List.flatten(&1, &2)` equals `fn(list, tail) -> List.flatten(list, tail) end`
 - Default arguments, can also be statements (evaluated on call)
   ```
   def fun(x \\ IO.puts "hello world) do
     x
   end
   ```
 - `Steam` for lazy enumeration, `Enum` for eager
 - `|>` for pipe: `[1,2,3] |> Enum.sum == 6`, equal to `Enum.sum([1,2,3]) == 6`
