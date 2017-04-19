# Elixir - Traversing an AST

Today I needed to traverse an Elixir AST and replace all occurences of a pinned variable with a value.

To briefly summarize what Elixir ASTs are:

Internally, Elixir represents Elixir code as a nested Elixir term. Most things are represented by a 3-tuple:
`{name, context, args}`. As an example the Elixir code `sum(1, 2)` is represented as `{:sum, [], [1, 2]}`.

This structure can be nested. E.g. `sum(1, 2 - 3)` becomes
`{:sum, [], [1, {:-, [context: Elixir, import: Kernel], [2, 3]}]}`.

Some Elixir terms are so simple, that they are just represented by itself.

To see the AST for an expression use `quote`:

```elixir
iex(1)> quote do: sum(1, 2)
{:sum, [], [1, 2]}
```

## Traversing

Ok. So the docs says that in the tuple the first item is an atom or another tuple. The second is a list with context
and the third is a list of values or an atom.

Also, lists and two-tuples are represented as themself where the inner values might be represented as a tuple.

I began to write function clauses for these different cases and ended up with something like this:

```elixir
def replace_pinned_var({:^, _context, [{name, _context, atom}]}) when is_atom(atom) do
  # This is the place I care about, since it represents a pinned variable.
  # Return a value based on some logic
  Macro.escape(some_value)
end
def replace_pinned_var({name, context, args}) when is_list(args) do
  {
    replace_pinned_var(name),
    context,
    replace_pinned_var(args)
  }
end
def replace_pinned_var(list) when is_list(list) do
  list |> Enum.map(&replace_pinned_var/1)
end
def replace_pinned_var({item_1, item_2}) do
  {
    replace_pinned_var(item_1),
    replace_pinned_var(item_2)
  }
end
def replace_pinned_var(term) do
  term
end
```

Most of this code is just traversing the AST. Only the first function clause holds my actual business logic.
It turns out there is a built-in helper function in Elixir to do the traversing part :)

`Macro.traverse/4` does exactly this. The docs are a little light on this function. This is the full documentation:

> Performs a depth-first traversal of quoted expressions using an accumulator.

It takes as arguments an AST, an accumulator, a function applied on the search down the tree and a function applied on
the search up the tree.

To visualize it, try this in iex:

```elixir
iex(4)> ast = quote do: 4 + 5 / 9
{:+, [context: Elixir, import: Kernel],
 [4, {:/, [context: Elixir, import: Kernel], [5, 9]}]}
iex(6)> pre = fn ast, acc ->
...(6)> IO.inspect(ast, label: "pre")
...(6)> {ast, acc}
...(6)> end
#Function<12.118419387/2 in :erl_eval.expr/5>
iex(7)> post = fn ast, acc ->
...(7)> IO.inspect(ast, label: "post")
...(7)> {ast, acc}
...(7)> end
#Function<12.118419387/2 in :erl_eval.expr/5>
iex(8)> Macro.traverse(ast, :acc, pre, post)
pre: {:+, [context: Elixir, import: Kernel],
 [4, {:/, [context: Elixir, import: Kernel], [5, 9]}]}
pre: 4
post: 4
pre: {:/, [context: Elixir, import: Kernel], [5, 9]}
pre: 5
post: 5
pre: 9
post: 9
post: {:/, [context: Elixir, import: Kernel], [5, 9]}
post: {:+, [context: Elixir, import: Kernel],
 [4, {:/, [context: Elixir, import: Kernel], [5, 9]}]}
{{:+, [context: Elixir, import: Kernel],
  [4, {:/, [context: Elixir, import: Kernel], [5, 9]}]}, :acc}
```

It might take a little while to read it through, but it gives a good understanding of `Macro.traverse/4`.

Using this function, I can rewrite my code to this:

```elixir
def replace_pinned_var(ast) do
  pre = fn
    {:^, _context, [{name, _context, atom}]}, _acc when is_atom(atom) -> {Macro.escape(some_value), nil}
    ast, _acc -> {ast, nil}
  end
  post = fn ast, _acc -> {ast, nil} end
  {ast, _acc} = Macro.traverse(ast, nil, pre, post)
  ast
end
```

Note that I don't need to accumulator, so I just use nils instead.
