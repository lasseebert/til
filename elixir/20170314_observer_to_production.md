# Elixir - Running observer on a production node

One of the most amazing things when first starting out with Elixir is the
built-in Erlang tool `:observer`.

If you don't know observer, [read more about it here](http://elixir-lang.org/getting-started/mix-otp/supervisor-and-application.html#observer).

It is simple enough to start an observer on a locally running application.
However, I had some trouble when I wanted to start the observer for my
production environment.

This guide will take you through the things I needed to do to make it work, step by step.

I am deploying with Distillery, and one of the steps here is specific to that,
but it should be easy to translate to whatever deployment flow you use.

## Firewall, cookie and node name

I have a firewall enabled on my production site, so I need to open up some ports.
It is not straight forward which ports to open, though.

Erlang requires two ports for connecting Nodes: One for discovery and one for the actual connection.

The discovery port defaults to 4369, but the connection port is choosen at random in a huge range.
I don't want to completely open my firewall, so I need to decrease the number of ports that can be chosen as
connection port.

This can be done by specifying some options to the VM.

```
-kernel inet_dist_listen_min 9000 -kernel inet_dist_listen_max 9000
```

This will lock the range to 9000..9000, which will only allow one port. This is ok for me, since I only expect to
connect to my production application from one machine at a time.

To specify this in Distillery I need to first add a vm_args file in which we also specify the node name and the cookie:

```
# rel/vm_args
-name <%= release_name %>@<%= server_name %>

-setcookie <%= cookie %>

-kernel inet_dist_listen_min 9000
-kernel inet_dist_listen_max 9000
```

Then in the release config set the `vm_args` attribute to point to this file.
Also set an `overlay_vars` which are the variables exposed in the vm_args file:

```
set vm_args: "rel/vm_args"
set overlay_vars: [
  server_name: "my_site.com",
  cookie: Application.fetch_env!(:my_app, :cookie)
]
```

And remember to open port 4369 and 9000 on the server.

## Connect to the remote node

Start a local iex session where you specify a name and the same cookie as used in production:

```
iex --name debug@127.0.0.1 --cookie mysupersecret cookie
```

Then connect to the node and start observer:

```
iex(debug@127.0.0.1)1> Node.connect(:"my_app@my_site.com")
iex(debug@127.0.0.1)2> :observer.start
```

In the "Nodes" menu, you should now be able to choose the production server.
