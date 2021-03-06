---
chapter: Advanced
title: Flat Style Setup
---

Block style setup is suitable for simple, quick'n'dirty scripts that need to
access databases, in a typical application setup, you want to break down
individual component definitions, like relations or commands, into separate
files.

> ROM & Frameworks
>
> Framework integrations **take care of the setup for you**. If you want to use ROM
> with a framework, please refer to specific instructions under [Getting Started](/learn/getting-started)
> section

## Setup

To do setup in flat style, create a `ROM::Configuration` object. This is the
same object that gets yielded into your block in block-style setup, so the API
is identical.

```ruby
configuration = ROM::Configuration.new(:memory, 'memory://test')
configuration.relation(:users)
# ... etc
```

When you’re finished configuring, pass the configuration object to
`ROM.container` to generate the finalized container. There are no differences in
the internal semantics between block-style and flat-style setup.

### Registering Components

ROM components need to be registered with the ROM environment in order to be
used.

```ruby
configuration = ROM::Configuration.new(:memory, 'memory://test')

# Declare Relations, Commands, and Mappers here
```

If you prefer to create explicit classes for your components you must register
them with the configuration directly:

```ruby
configuration = ROM::Configuration.new(:memory, 'memory://test')

configuration.register_relation(OneOfMyRelations)
configuration.register_relation(AnotherOfMyRelations)
configuration.register_command(User::CreateCommand)
configuration.register_mapper(User::UserMapper)
```

You can pass multiple components to each `register` call, as a list of
arguments.

### Auto-registration

ROM provides `auto_registration` as a convenience method for automatically
`require`-ing and registering components that are not declared with the DSL. At
a minimum, `auto_registration` requires a base directory. By default, it will
load relations from `<base>/relations`, commands from `<base>/commands`, and
mappers from `<base>/mappers`.

```ruby
configuration = ROM::Configuration.new(:memory)
configuration.auto_registration(__dir__)
container = ROM.container(configuration)
```

## Relations

While the DSL syntax is often convenient, Relations can also be defined with a
class extending `ROM::Relation` from the appropriate adapter.

```ruby
# Defines a Users relation for the SQL adapter
class Users < ROM::Relation[:sql]

end

# Defines a Posts relation for the HTTP adapter
class Posts < ROM::Relation[:http]

end
```

Relations can declare the specific
[gateway](http://rom-rb.org/learn/glossary#gateway) and
[dataset](http://rom-rb.org/introduction/glossary/#dataset) it takes data from,
as well as the registered name of the relation. The following example sets the
default options explicitly:

```ruby
class Users < ROM::Relation[:sql]
  register_as :users    # the registered name; eg. for use in Repository’s relations(...) method
  gateway :default      # the gateway name, as defined in setup
  dataset :users        # eg. in sql, this is the table name
end
```

## Commands

Just like Relations, Commands have an alternative style as a regular class:

```ruby
class CreateUser < ROM::Commands::Create[:memory]

end
```

Commands have three settings: their relation, which takes the registered name of
a relation; their result type, either `:one` or `:many`; and their registered
name.

```ruby
class CreateUser < ROM::Commands::Create[:memory]
   register_as :create
   relation :users
   result :one
end
```

> Typically, you're going to use [repository command interface](/learn/repositories/quick-start);
> custom command classes are useful when the built-in command support in
> repositories doesn't meet your requirements
