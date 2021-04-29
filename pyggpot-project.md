
# Table of Contents

1.  [Introduction](#orgade5dac)
    1.  [First, A Spec](#orga60c57c)
2.  [Step 1: Build Pyggpot](#orgcc8a2e7)
3.  [Step 2: Understand what you've built](#orga0be641)
    1.  [Walkthrough](#orgc8ea7dd)
        1.  [Schema](#org2b4ac4b)
        2.  [Service contract, aka Protobuf schema](#org8e3a8ee)
        3.  [Service providers](#orgbb11542)
        4.  [Where is main()?](#org50e351e)
4.  [The Assignment](#org3e123e6)
    1.  [Fix a validation error message](#org388cda8)
    2.  [Code the RemoveCoins service](#org1ee8829)
    3.  [Document a better design to support the RemoveCoins service](#org7219567)
5.  [I'm Done!](#org49a9de8)


<a id="orgade5dac"></a>

# Introduction

The point of this example project assignment is to challenge you with
the kind of code and design challenges we developers face daily. A
common first step is figuring out some else's code, getting it to
work, and fixing or adding features.

While there's no hard time limit, we think this project will take most
developers with some Golang experience 45 to 90 minutes. We certainly
don't want to tie you up for more than 90 minutes, unless it's for
your own extracurricular purposes, like learning or refreshing your
Golang knowledge. We're not hung up on how long you take however, but
rather on the depth of your thinking.


<a id="orga60c57c"></a>

## First, A Spec

The Pyggpot service at <https://github.com/aspiration-labs/pyggpot>
emulates a piggy bank. The service supports the following actions

-   Create a Pot with a name and a capacity of some number of coins
-   Add gold, silver, and/or bronze Coins to the Pot. All coins are
    the same size. Gold are worth 100, silver 10, and bronze 1.
-   Remove some number of Coins from the Pot. You shake the Pot upside
    down and Coins come out in a uniform random distribution.

A pot is opaque, so it's not possible to see inside how many or what
kind of coins are in a pot. Plus, we have a very bad memory, and
always forget what we put in the pot.


<a id="orgcc8a2e7"></a>

# Step 1: Build Pyggpot

The top level README.md in <https://github.com/aspiration-labs/pyggpot>
should cover building and running the code. OSX and Ubuntu Linux are
currently supported. Most other Linuxes should work, but haven't been
tested. Windows is not yet supported.

If you get to the point of a working Swagger site at
<http://localhost:8080/swaggerui/> you're ready for the next step. If
you believe there's a bug in the build, feel free to email with repro
details.


<a id="orga0be641"></a>

# Step 2: Understand what you've built

You should be able to work through this assignment without significant
knowledge of the tooling details. If you have worked with [Google
Protocol Buffers](https://developers.google.com/protocol-buffers/), [Twirp](https://github.com/twitchtv/twirp), and/or [xo](https://github.com/xo/xo/) then you're way ahead of the
game. However, the makefile should take care of that machinery, and
you just need to understand some sql, regular expressions, and go code
to push-on through.


<a id="orgc8ea7dd"></a>

## Walkthrough

Usually when you get thrown into a new code, you get some kind of
walkthrough from another developer. Sometimes it's even
helpful. That's the intent of this section.


<a id="org2b4ac4b"></a>

### Schema

Peruse `sql/schema.sqlite3.sql`. It's a simple two table schema, many
coins to each pot. A Coin record is a count of coins of a
denomination. As designed, the Coin table may have multiple records
for a given denomination.

If in your process you want to modify the schema or extend the xo
model processing, that's fine (though not required by the questions
below.) Be aware that you'll need to reset the database, `make
resetdb` if you change schema, and rerun `make models` in either case.


<a id="org8e3a8ee"></a>

### Service contract, aka Protobuf schema

Look at `proto/coin/service.proto` and
`proto/pot/service.proto`. These two specs define our service API,
request and response formats. The `message` schema will be compiled
into native Golang data types, which is all you really need to focus
on.

After building you'll find the generated code in `rpc/go/coin` and
`rpc/go/pot`. You don't need to fully grok all the generated code, but
worth knowing that the Go data types for protobuf messages are in the
`service.pb.go` files. The `service.twirp.go` holds the interface spec
that needs to be implemented for the service.

Again, if in improving the design or just experimenting you do change
protobuf schemas you'll need to rerun `make services`.


<a id="orgbb11542"></a>

### Service providers

For this project we refer to our service implementations as
"providers", which you'll find in `internal/providers/...`  You can
review the pot and coin provider in the `provider.go` files. You
should note, for example, a `type Pot interface` from
rpc/go/pot/service.twirp.go is implemented here for a
`potServer`. This interface implementation, `potServer` in this case,
also holds useful state data for the service, like database
connections.

Similarly, `type Coin interface` has an implementation in
`internal/providers/coin/provider.go` that you'll spend some time
in. It's in scope to rewrite any existing code in that implementation
if it fits with your plan of attack.


<a id="org50e351e"></a>

### Where is main()?

The service startup is in `cmd/server/main.go`


<a id="org3e123e6"></a>

# The Assignment

At this point you should have Pyggpot built and running, and perhaps
have thrown a few curls at it directly or via the swaggerui. Here's
your todo list.


<a id="org388cda8"></a>

## TODO Fix a validation error message

If I

    curl -X POST "http://localhost:8080/twirp/pyggpot.pot.Pot/CreatePot" \
      -H "accept: application/json" -H "Content-Type: application/json" \
      -d "{ \"pot_name\": \"PP\", \"max_coins\": 10}"

I get a 400 status back with response

    {
      "code": "invalid_argument",
      "msg": "invalid field PotName: Can contain only alphanumeric characters, dot and underscore. ",
      "meta": {
        "argument": "invalid field PotName: Can contain only alphanumeric characters, dot and underscore."
      }
    }

Similarly, `pot_name` of "PP." and "PP-", while "PPP" return 200 status
and a new Pot.

Your task: find the validation message and the rule behind it; write
an accurate error message that will avoid end user rage clicking.


<a id="org1ee8829"></a>

## TODO Code the RemoveCoins service

In `internal/providers/coin/provider.go` the `RemoveCoins` function is
unimplemented. The spec is like shaking a piggy bank upside down until
coins come out. The type (denomination) of the coin is random, based
on the proportion of that coin type in the pot.

The `proto/coin/service.proto` response spec
(`coin_service.CoinsListResponse` in `provider.go`) is the list of
coins and denomations that slipped out of the Pyggpot.  denominated
coins.

IMPORTANT NOTE: In this exercise you do NOT have to deal with
concurrent calls or race conditions in database access. Assume only
one single threaded instance of daemon will be running.


<a id="org7219567"></a>

## TODO Document a better design to support the RemoveCoins service

We (royal) have decided that the given design of coins in pots is not
that great. In particular, the `RemoveCoins` service may be doing more
work than we'd like to retrieve coins randomly. We would also like to
scale, with concurrent Pyggpot processes.

Write a short redesign proposal (think refactor ticket for another
developer) which may include schema, service spec, or provider
implementation changes. You don't have to write the code for this, but
justify your redesign based on simplicity, performance, or any other
positive you find compelling. However, if you did refactor schema or
existing code, give the motivation for that refactor here.


<a id="org49a9de8"></a>

# I'm Done!

The easist way to respond is 

    git add . && git diff --cached >MyPyggpotMods.diff

and email the diff file back to whoever sent you these instructions.

Tia!

