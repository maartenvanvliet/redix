# Redix

[![Build Status](https://travis-ci.org/whatyouhide/redix.svg?branch=master)](https://travis-ci.org/whatyouhide/redix)
[![Hex.pm](https://img.shields.io/hexpm/v/redix.svg)](https://hex.pm/packages/redix)

> Fast, pipelined, resilient Redis client for Elixir.

![Cover image](http://i.imgur.com/ZG2RXsb.png)

Redix is a [Redis][redis] client written in pure Elixir with focus on speed, correctness, and resiliency (that is, being able to automatically reconnect to Redis in case of network errors).

This README refers to the `master` branch of Redix, not the latest released version on Hex. Make sure to checkt [the documentation](http://hexdocs.pm/redix) for the version you're using.

## Installation

Add the `:redix` dependency to your `mix.exs` file:

```elixir
defp deps() do
  [{:redix, ">= 0.0.0"}]
end
```

Then, run `mix deps.get` in your shell to fetch the new dependency.

## Usage

Redix is simple: it doesn't wrap Redis commands with Elixir functions. It only provides functions to send any Redis command to the Redis server. A Redis *command* is expressed as a list of strings making up the command and its arguments.

Connections are started via `start_link/0,1,2`:

```elixir
{:ok, conn} = Redix.start_link(host: "example.com", port: 5000)
{:ok, conn} = Redix.start_link("redis://localhost:6379/3", name: :redix)
```

Commands can be sent using `Redix.command/2,3`:

```elixir
Redix.command(conn, ["SET", "mykey", "foo"])
#=> {:ok, "OK"}
Redix.command(conn, ["GET", "mykey"])
#=> {:ok, "foo"}
```

Pipelines are just lists of commands sent all at once to Redis for which Redis replies with a list of responses. They can be used in Redix via `Redix.pipeline/2,3`:

```elixir
Redix.pipeline(conn, [["INCR", "foo"], ["INCR", "foo"], ["INCRBY", "foo", "2"]])
#=> {:ok, [1, 2, 4]}
```

`Redix.command/2,3` and `Redix.pipeline/2,3` always return `{:ok, result}` or `{:error, reason}`. If you want to access the result directly and raise in case there's an error, bang! variants are provided:

```elixir
Redix.command!(conn, ["PING"])
#=> "PONG"

Redix.pipeline!(conn, [["SET", "mykey", "foo"], ["GET", "mykey"]])
#=> ["OK", "foo"]
```

#### Resiliency

Redix is resilient against network errors. For example, if the connection to Redis drops, Redix will automatically try to reconnect periodically at a given "backoff" interval. Look at the documentation for the `Redix` module and at the ["Reconnections" page][docs-reconnections] in the documentation for more information on the available options and on the exact reconnection behaviour.

#### Pub/Sub

Redix doesn't support the Pub/Sub features of Redis. For that, there's [`redix_pubsub`][redix-pubsub].

## Using Redix in the Real World™

Redix is low-level, but it's still built to handle most things thrown at it. For many applications, you can avoid pooling with little to no impact on performance. Read the ["Real world usage" page][docs-real-world-usage] in the documentation for more information on this and pooling strategies that work better with Redix.

## Contributing

Clone the repository and run `$ mix test` to make sure everything is working. For tests to pass, you must have a Redis server running on `localhost`, port `6379`. Both may be configured using the environment variables `REDIX_TEST_HOST` and `REDIX_TEST_PORT` respectively. **Note**: tests will wipe clean all the databases on the running Redis server, as they call `FLUSHALL` multiple times.

## License

Redix is released under the MIT license. See the [license file](LICENSE.txt).


[redis]: http://redis.io
[redix-pubsub]: https://github.com/whatyouhide/redix_pubsub
[docs-reconnections]: http://hexdocs.pm/redix/reconnections.html
[docs-real-world-usage]: http://hexdocs.pm/redix/real-world-usage.html
