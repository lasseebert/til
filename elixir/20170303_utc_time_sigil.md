# Elixir - UTC DateTime sigil in tests

I like UTC timestamps. Whenever I save the current time or another absolute
timestamp, I always save it in the database as UTC.

I then sometimes need to produce UTC timestamps in my tests in order to setup
a test case. This is not much fun.

The simplest way to do this with the standard library is something like:

```elixir
~N[2017-03-03 14:21:15] |> DateTime.from_naive!("Etc/UTC")
```

Yuk! This quickly demotivates me to write tests.

There must be a better way. And there is :)

One can simply implement a custom sigil and import it in the base test case module.

```elixir
defmodule MyApp.Test.Sigils do
  @moduledoc """
  Sigils to ease testing
  """

  @doc """
  Creates a UTC DateTime
  """
  defmacro sigil_U({:<<>>, _, [string]}, []) do
    string
    |> NaiveDateTime.from_iso8601!
    |> DateTime.from_naive!("Etc/UTC")
    |> Macro.escape
  end
end
```

This allows me to use `~U[2017-03-03 14:21:15]` to get a UTC DateTime from my tests.

Nice!
