# Elixir - The Access module and nils

Today I was surprised about the behaviour of the Access module syntax `map[key]`.

I ran a test that asserted something about a json response. Something like this:

```elixir
assert body["data"]["relationships"]["user"]["data"]["id"] == 42
```

The assertion failed with a message like this:

```
Left: nil
Right 42
```

Hmm.

Why is the `id` of the `user` not set?

Somehow I was sure that if the entire `user` was not set, the left expression would raise an error, since I was
otherwise trying to get a value by key from `nil`, which does not make sense.

Then I tried stuff in the console:

```
$ iex
Erlang/OTP 19 [erts-8.2] [source] [64-bit] [smp:8:8] [async-threads:10] [hipe] [kernel-poll:false]

Interactive Elixir (1.4.2) - press Ctrl+C to exit (type h() ENTER for help)
iex(1)> a = %{}
%{}
iex(2)> a[:a]
nil                 <-- This is expected
iex(3)> a[:a][:b]
nil                 <-- This surprised me
iex(4)> nil[:a]
nil                 <-- WTF is going on??!!!one1!!!
```

I went investigating. I knew about the Access module, but had never studied it before. So I went straight to the source
code.

It seems that the `[]` syntax calls the `Access.get/3` function with the third `default` defaulting to `nil`.
`Access.get/3` calls `Access.fetch/2` function, which is defined to work on:

* Structs
* Maps
* Keyword lists
* ...and `nil` (returning `:error` which makes `Access.get/3` return the default of `nil`)

So this is why `nil[:a] == nil`.

## Lesson learned

Use the more explicit `Map.get/3` when explicitness is needed:

```
iex(5)> Map.get(nil, :a)
** (BadMapError) expected a map, got: nil
    (stdlib) :maps.find(:a, nil)
    (elixir) lib/map.ex:234: Map.get/3
```

I will continue using the Access syntax in my tests, since I just need to assert that the entire expression is some
value. But beware when asserting that something is nil. E.g.

```elixir
assert foo[:bar][:baz] == nil
```

This will not tell you much. Something like this would work better

```elixir
assert foo |> Map.fetch!(:bar) |> Map.fetch!(:baz) == nil
```
