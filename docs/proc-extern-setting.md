# Proc External Implementation Setting

| Lead Author(s) | Implemented | GitHub Links |
|---|---|---|
| distributivgesetz | :x: No | TBD |

## Overview

This document proposes an alternative method of binding external implementations of code into DM codebases with the help of two new proc settings: `extern_lib` and `extern_func`.

## Motivation

The current method of calling external functions through `call_ext` feels very disordered. In order to bring external calls in line with modern practices of writing DM code, many codebases opt to hide `call_ext` syntax behind macros, which leads to macro pollution. The syntax also cannot account for the type of the input parameters, making static type checking impossible in this regard.

## Design

The proposal includes two new proc settings:

- `extern_lib`
  This setting holds to the name of the library the function was defined in. The same name schema as `call_ext` is used here.
- `extern_name`
  This setting holds the name of the exported function in the library.
  
Example:

```
/proc/foo_bar_extern()
    set extern_lib = "foo" // points to libfoo.so or foo.dll
    set extern_name = "bar" // points to exported function named "bar"
    
    // this is a stub proc, any proc implementation will be ignored by the compiler
    return 
```

Defining these in any proc will inform the compiler and runtime to treat the proc as externally implemented.

- When using either setting, the other setting must also be defined, otherwise a compiler error is raised. 
- Only global procs may have these settings. If a setting is set on an instance proc, a compiler error is raised.
- Externally defined procs are not allowed to be used as verbs, otherwise a compiler or runtime error is raised.
- When a proc is defined as external, the proc is not allowed to have code in its body, otherwise a compiler warning/error is raised.

```
/proc/foo_bar() // compiler error, incomplete extern proc definition
    //set extern_lib = "foo"
    set extern_name = "bar"
    
/datum/proc/foo_bar() // compiler error, instance proc defined as extern
    set extern_lib = "foo"
    set extern_name = "bar"

/verb/foo_bar() // compiler error, verb defined as extern
    set extern_lib = "foo"
    set extern_name = "bar"
    
/verb/foo_bar()
    set extern_lib = "foo"
    set extern_name = "bar"
    
    return 1 // compiler warning, code here is ignored
```

The proc arguments may map to the arguments of the external function. The types of the arguments and the return value can be defined in order to facilitate static type checking.

DM library maintainers may choose to use this syntax as an alternative to `call_ext`. Practically this only allows for static type checking, and less macro pollution if a maintainer wishes to only target OpenDream with their library.

<!-- This is the bulk of the proposal. Explain the design in enough detail for somebody familiar with the language to understand, and include examples of how the feature is used. Also describe how codebases operating on BYOND can still utilize or ignore the feature as they see fit. -->

## Considerations & Drawbacks

In order to keep parity with BYOND, library maintainers opting to use this feature would either need to obscure extern procs behind a macro, or they would need to add a conditional codeblock inside the proc that includes `call_ext`. Performance in this regard would stay the same to `call_ext` or be worse due to proc caling overhead when keeping the proc in BYOND.

This is optional developer facing syntax, therefore no impact on users or developers migrating to OpenDream with this feature.
