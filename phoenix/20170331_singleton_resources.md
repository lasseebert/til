# Phoenix - Singleton resources

In Phoenix one defines routes in the `Router`.

Sometimes I need to define a resource route but without the `:id` part.

A common example of this is a `/profile`, which shows a private profile page for the currently signed in user.

One could imagine both a `GET /profile` and a `PATCH /profile`.

This can be done using `resources` which normally would expect an id at the end of the path, but with `singleton: true`
this is changed to a set of resource routes without an id.

To define the routes above as a singleton resource do this:

```elixir
resources "/profile", ProfileController, only: [:show, :update], singleton: true
```

Singleton routes are also commonly used as nested routes.

Imagine a user resource that has-one `UserSettings` which can be showed and updated:

```elixir
resources "/users", UserController do
  resources "/settings", UserSettingsController, only: [:show, :update], singleton: true
end
```

This will generate the router paths:

```
GET /users/:user_id/settings
PATCH /users/:user_id/settings
```

Notice that the `:id` changed to `:user_id`, which will also be the name of the user id key in the `params`.
