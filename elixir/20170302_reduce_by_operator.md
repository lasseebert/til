# Elixir - Reduce by operator

I already knew that `+` is a shortcut to `Kernel.+/2`:

```elixir
iex(1)> 1 + 2
3

iex(2)> Kernel.+(1, 2)
3
```

But I didn't know that you can skip the `Kernel`-part when referencing the function with the `&`-notiation.

This makes reducing by an operator very nice:

```elixir
iex(3)> &Kernel.+/2
&:erlang.+/2

iex(4)> &+/2
&:erlang.+/2

iex(5)> [1, 2, 3] |> Enum.reduce(&+/2)
6
```
