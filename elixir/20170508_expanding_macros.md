# Elixir - Expanding macros

Today I learned a thing about macros and how they expand.

I thought that the Elixir AST at compile time would expand any macros until only non-macro code was left.

But I was wrong. Elixir will only expand nested macros, when the inner most macro call is the body of the outer macro.

As an example look at this code:

```elixir
defmodule Macros do
  defmacro delegate_to_forty_two do
    quote do
      Macros.forty_two
    end
  end

  defmacro forty_two do
    quote do
      42
    end
  end

  defmacro does_not_fully_expand do
    quote do
      Macros.forty_two + Macros.forty_two
    end
  end
end
```

We can use `Macro.expand/2` and `Macro.to_string/1` to see how the macros expand:

```elixir
iex(4)> quote(do: Macros.delegate_to_forty_two) |> Macro.expand(__ENV__) |> Macro.to_string
"42"
```

This fully expands to `42`. When trying with `Macros.does_not_fully_expand` in which `:+` is the body of the quoted
expression, we will get another result:

```elixir
iex(5)> quote(do: Macros.does_not_fully_expand) |> Macro.expand(__ENV__) |> Macro.to_string
"Macros.forty_two() + Macros.forty_two()"
```

I would have expected this to expand to `42 + 42`, but I was wrong and I learned something new today.
