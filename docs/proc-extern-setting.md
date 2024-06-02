# Proc External Implementation Setting

| Lead Author(s) | Implemented | GitHub Links |
|---|---|---|
| distributivgesetz | :x: No | TBD |

## Overview

This document proposes an alternative method of binding external implementations of code into DM codebases with the help of a new proc setting: `opendream_extern_impl`.

## Motivation

The current method of calling external functions through `call_ext` feels very clunky. In order to bring external calls in line with modern practices of writing DM code, many codebases opt to hide `call_ext` syntax behind macros. Also, `call_ext` syntax cannot account for the type of the input parameters, making static type checking impossible in this regard.

Additionally, since external functions cannot be determined at compile time with `call_ext`, ahead-of-time or just-in-time optimizations such as external call inlining cannot be implemented, because `call_ext` implies late-bound external calls.

Overall, the main motivation of `opendream_extern_impl` is to provide neater and more robust syntax for external calls, as well as the ability for the compiler to determine which external functions an OpenDream executable will use, to allow for external call optimizations.

## Design

The proposal includes a new proc setting:

- `opendream_extern_impl`
  This defines the name and library of the external function, stored in a list tuple. The same naming schema as `call_ext` is used here.
  
Example:

```
/proc/foo_bar_extern()
    set opendream_extern_impl = list("foo", "bar") // points to a function called bar in libfoo.so or foo.dll
    // this is a stub proc, any proc implementation will be ignored by the compiler
```

Defining these in any proc will inform the compiler and runtime to treat the proc as externally implemented.

- Only global procs may have these settings. If a setting is set on an instance proc, a compiler error is raised.
- Externally defined procs are not allowed to be used as verbs, otherwise a compiler or runtime error is raised.
- When a proc is defined as external, the proc is not allowed to have code in its body, otherwise a compiler error is raised.

```
/datum/proc/foo_bar() // compiler error, instance proc defined as extern
    set opendream_extern_impl = list("foo", "bar")

/verb/foo_bar() // compiler error, verb defined as extern
    set opendream_extern_impl = list("foo", "bar")

/mob/New()
    verbs += /proc/foobar // runtime error, cannot add extern proc as verb

/verb/foo_bar()
    set opendream_extern_impl = list("foo", "bar")
    
    return 1 // compiler error, code here is ignored
```

The proc arguments should map to the arguments of the external function. The types of the arguments and the return value can be defined in order to facilitate static type checking.

DM library maintainers may choose to use this syntax as an alternative to `call_ext`. At compile time this allows for static type checking and less macro pollution if a maintainer wishes to only target OpenDream with their code.

## Considerations & Drawbacks

In order to keep parity with BYOND, library maintainers opting to use this feature would either need to obscure extern procs behind a macro, or they would need to add a conditional codeblock inside the proc that includes `call_ext`. Performance would stay the same or worse compared to `call_ext` in BYOND due to proc call overhead when moving the external call behind a proc .

This is optional developer facing syntax, therefore there is no impact on users or developers migrating to OpenDream with this feature.
