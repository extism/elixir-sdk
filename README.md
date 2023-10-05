# Extism

Extism Host SDK for Elixir and Erlang

> **Note**: This is an early release for 1.0 and is not yet ready. Use a 0.x release from the [extism/extism](https://github.com/extism/extism/tree/main/elixir) repo for now.

## Docs

Read the [docs on hexdocs.pm](https://hexdocs.pm/extism/).

## Installation

You can find this package on [hex.pm](https://hex.pm/packages/extism).

```elixir
def deps do
  [
    {:extism, "1.0.0-rc1"}
  ]
end
```

> **Note**: You'll need a rust toolchain installed to build this package. See [Install Rust](https://www.rust-lang.org/tools/install) to install for your platform.

## Getting Started

This guide should walk you through some of the concepts in Extism and this ruby library.

> *Note*: You should be able to follow this guide by copy pasting the code into `iex` using `iex -S mix`.

### Creating A Plug-in

The primary concept in Extism is the [plug-in](https://extism.org/docs/concepts/plug-in). You can think of a plug-in as a code module stored in a `.wasm` file.

Since you may not have an Extism plug-in on hand to test, let's load a demo plug-in from the web:

```elixir
url = "https://github.com/extism/plugins/releases/latest/download/count_vowels.wasm"
manifest = %{wasm: [%{url: url}]}
{:ok, plugin} = Extism.Plugin.new(manifest, false)
```

> **Note**: See [the Manifest docs](https://extism.github.io/ruby-sdk/Extism/Manifest.html) as it has a rich schema and a lot of options.

### Calling A Plug-in's Exports

This plug-in was written in Rust and it does one thing, it counts vowels in a string. As such, it exposes one "export" function: `count_vowels`. We can call exports using [Extism.Plugin#call/3](https://hexdocs.pm/extism/Extism.Plugin.html#call/3):

```ruby
{:ok, output} = Extism.Plugin.call(plugin, "count_vowels", "Hello, World!")
# => {"count": 3, "total": 3, "vowels": "aeiouAEIOU"}
```

All exports have a simple interface of bytes-in and bytes-out. This plug-in happens to take a string and return a JSON encoded string with a report of results.

### Plug-in State

Plug-ins may be stateful or stateless. Plug-ins can maintain state b/w calls by the use of variables. Our count vowels plug-in remembers the total number of vowels it's ever counted in the "total" key in the result. You can see this by making subsequent calls to the export:

```ruby
{:ok, output} = Extism.Plugin.call(plugin, "count_vowels", "Hello, World!")
# => {"count": 3, "total": 6, "vowels": "aeiouAEIOU"}
{:ok, output} = Extism.Plugin.call(plugin, "count_vowels", "Hello, World!")
# => {"count": 3, "total": 9, "vowels": "aeiouAEIOU"}
```

These variables will persist until this plug-in is freed or you initialize a new one.

### Configuration

> TODO: Implement config

Plug-ins may optionally take a configuration object. This is a static way to configure the plug-in. Our count-vowels plugin takes an optional configuration to change out which characters are considered vowels. Example:

```ruby
{:ok, output} = Extism.Plugin.call(plugin, "count_vowels", "Yellow, World!")
# => {"count": 3, "total": 3, "vowels": "aeiouAEIOU"}

{:ok, plugin} = Extism.Plugin.new(manifest, false, %{"vowels": "aeiouyAEIOUY"})
{:ok, output} = Extism.Plugin.call(plugin, "count_vowels", "Yellow, World!")
plugin.call("count_vowels", "Yellow, World!")
# => {"count": 4, "total": 4, "vowels": "aeiouAEIOUY"}
```

### Host Functions

We don't offer host function support in this library yet. If it is something you need, please [file an issue](https://github.com/extism/elixir-sdk/issues/new)!

