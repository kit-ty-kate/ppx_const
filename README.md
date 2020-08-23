ppx_const
=========

This is an OCaml language extension implementing `if%const` and `match%const` statements. The `if%const` and `match%const` are evaluated at compile time, and the appropriate clause substituted without the ignored clause(s) being fully compiled. This allows you to avoid consequences such as module inclusion or type inference changes which would otherwise have resulted from the ignored clause(s).

In other words, ppx\_const works like `#if` in the C preprocessor, but is implemented entirely within the OCaml language using the [ppx](http://whitequark.org/blog/2014/04/16/a-guide-to-extension-points-in-ocaml/) mechanism. In conjunction with [ppx_getenv](https://github.com/whitequark/ppx_getenv), this provides a lightweight alternative to Cppo.

This software was written by Andi McClure <<andi.m.mcclure@gmail.com>>, based on whitequark's ppx\_getenv sample. Significant upgrades were contributed by Kitty Kate.

Because ppx\_const is based on ppx, it requires OCaml 4.02 or newer.

Usage
-----

ppx\_const may be invoked with either `if%const` or `match%const`.

### if%const

`if%const` may be invoked with either of:

    if%const COND then A else B
    if%const COND then A

COND must be one of the following:

* `true`
* `false`
* An expression consisting of two literals (strings, ints, floats) and either the `<>` or `=` operator.

COND may also contain extension nodes (including `if%const`s) as long as they evaluate to a constant expression by the time ppx\_const sees them.

A and B are not required to be of the same type. Like with normal `if`, the return type of `if%const false then X` is unit.

### match%const

ppx\_const can also be invoked with the following:

    match%const MATCHED with P\_1 -> E\_1 | ... | P\_n -> E\_n

MATCHED and P\_1..P\_n must be either literals (strings, ints, floats) or one of the special constructors `true` and `false`. Like with `if%const`, MATCHED may contain extension nodes, although the P1..P\_n may not.

The patterns P\_1..P\_n may also be variables or `_`, in which case they will always match. Matching on a variable name will "bind" a variable by that name in the match expression: If a pattern P\_i is a variable `x` then the expression, if matched, will compile to `let x = MATCHED in E_i`.


An example: Using ppx_const with ppx\_gentenv
---------------------------------------------

Say your program has a Graph module with heavyweight dependencies (cairo or whatever). Some users may prefer to compile your program without the graph feature, so that they don't have to install the dependencies. You can do this by installing ppx\_const and ppx\_getenv, and invoking the graph feature like this:

    if%const [%getenv "BUILD_OMIT_GRAPH"] = "" then
        Graph.create filename
    else
        print_endline "Graph feature not available."

In this example, when you build, if the `BUILD_OMIT_GRAPH` environment variable is set to a nonempty string then the `Graph.create` call will be omitted entirely from the compiled binary. If this is the only invocation of Graph, then the Graph module and all its dependencies will also be omitted from the binary. If you do not set this environment variable, the `[%getenv` check will become an empty string at build time and the graph function will be included.

**OCamlbuild users take note:** For this example to work, you'll need to make **certain** that ppx\_getenv runs before ppx\_const, so that `if%const` sees a constant string and not the `[%` node. If you are using Dune, then Dune will take care of this for you. In OCamlbuild, you set a load order by ordering the packages like this in your `_tags` file:

    <*>: package(ppx_getenv, ppx_const)

License
-------

[Creative Commons Zero](LICENSE.txt) ("public domain or equivalent")
