# Extended Collection Types

| Lead Author(s) | Implemented | GitHub Links |
|---|---|---|
| ike709 | :x: No | TBD |

## Overview

Currently, the only collection type in BYOND is the `list()` type, which doubles as both a dynamically resizable list and also as a dictionary. This RFC proposes exposing several additional collection types directly from C# to DM.

Note that typed collections (e.g. an array of text strings) will be implemented as a future RFC; this RFC primarily focuses on *which* collection types to add in the first place.

## Motivation

The implementation of BYOND lists is highly inefficient in OpenDream as we can't just back it with a C# list or a C# dictionary. Furthermore, other collection types would offer performant convenience features for developers that they do not need to implement themselves, such as performant hashsets or stacks.

## Design

The current `list()` type will remain unchanged for BYOND compatibility, with the other types being added *in addition to* these legacy lists.

The following collection types will essentially be exposed directly from C# to DM through a minimal wrapper API:

- **Array**: A fixed-length collection of elements.
- **Dictionary**: A collection of key-value pairs. Equivalent to existing associative lists, and will serve as the internal implementation of BYOND's planned `alist()` feature.
- **HashSet**: A collection that contains no duplicate elements.
- **ODList**: A dynamically resizable collection that is purely a list with none of the assoc-list functionality that BYOND lists have.
- **Queue**: A first-in first-out (FIFO) collection.
- **Stack**: A last-in first-out (LIFO) collection.

Generally speaking, the helper procs from `list()` will also operate on other collections where applicable (e.g. `list.Join()` and `array.Join()` would be equivalent). A few collections will implement additional helpers as noted below.

**Note:** Initializing OpenDream's collection types with a fixed size, for example `var/array/foo[10]`, will initialize it with a `capacity` of 10 but *will not* insert any `null` elements; the actual contents of the collection will be empty. This is **breaking behavior** from BYOND lists such as `var/list/L[10]`, which inserts 10 `null` elements into the list. BYOND lists in OpenDream will preserve this behavior for compatibility. Otherwise, see the "Collection Capacity API" subsection below for more information.

### Array

A fixed-length collection of elements. Almost identical to `list()` except it cannot be resized.

Example usage:
```
var/array/foo = array("a","b","c")
foo += "z" // Compile-time error, cannot resize arrays

world.log << foo.Join() // Prints "abc"

foo[1] = "z" // Valid modification
```

### Dictionary
A collection of key-value pairs. Equivalent to existing associative lists, and will serve as the internal implementation of BYOND's planned `alist()` feature.

Example usage:
```
var/dict/bar = array("a" = 1,"b" = 2,"c" = 3)
bar += "d" // Compile-time error, no value specified

bad["d"] = 4 // Valid modification

world.log << bar["c"] // Prints 3

world.log << bar["test"] // Runtime error, key not found

bar.contains_key("test") // Returns FALSE

bar.get_value("test") // Returns null, since the key doesn't exist

world.log << bar[1] // Runtime error, dicts can't be numerically indexed
```

`dict.keys` and `dict.values` will return a new `array()` of the dictionary's keys and values, respectively. Modifying these will not modify the existing dictionary.

`dict.contains_key(key)` will return `TRUE` or `FALSE` depending on if the `key` is present in the dictionary.

`dict.get_value(key)` will return the value of the specified `key` or `null` if the key is not present in the dictionary.

### HashSet
Essentially identical to a list, except it cannot contain duplicates. Inserting an element that is already present in the hashset is not an error and instead does nothing.

Direct insertion (e.g. `hashset += "foo"`) is supported, but `hashset.add(element)` will return `TRUE` or `FALSE` depending on if the element already existed in the hashset.

Example usage:
```
var/hashset/foo = hashset("a","b","c")

foo += "c" // Valid but nothing happens since `foo` already contains "c"

foo["c"] = 5 // Compile-time error, hashsets cannot be associative

world.log << foo.add("c") // Prints FALSE and inserts nothing since `foo` already contains "c"

world.log << foo.add("d") // Prints TRUE and inserts "d" into `foo`
world.log << foo.Join() // Prints "abcd"
```

### ODList 
Identical to BYOND `list()` but with no support for associative lists. This facilitates various internal optimizations over using `list()`.

Unlike BYOND lists, ODLists also implement the "Constant Collections" and "Collection Capacity API" defined in subsequent sections.

Example usage:
```
var/odlist/foo = odlist("a","b","c")

foo += "d" // Valid modification, just like a BYOND list

foo["c"] = 5 // Compile-time error, ODLists cannot be associative

world.log << foo.Join() // Prints "abcd"
```

### Queue
Similar to lists, except elements are first-in first-out. Most of the list API will remain, except for inserting and removing elements. These will use `queue.enqueue(element)` and `queue.dequeue()`.

Example usage:
```
var/queue/foo = queue("a","b","c")

foo += "d" // Compile-time error, must use foo.enqueue("d")

world.log << foo[2] // Prints "b", indexing is supported

world.log << foo.dequeue() // Removes "a" from `foo` and prints "a"
```

A `queue.to_array()` helper will also be provided, and returns the queue's elements as a shallow-copied `array()`.

### Stack 
Similar to queues, except elements are last-in first-out. Most of the list API will remain, except for inserting and removing elements. These will use `stack.push(element)` and `stack.pop()`.

Example usage:
```
var/stack/foo = stack("a","b","c")

foo += "d" // Compile-time error, must use foo.push("d")

world.log << foo[2] // Prints "b", indexing is supported

foo.push("d")
world.log << foo.pop() // Removes "d" from `foo` and prints "d"
```

A `stack.to_array()` helper will also be provided, and returns the stack's elements as a shallow-copied `array()`.

## Constant Collections

All OpenDream collections can now be declared as constant, which prevents any future modifications to the collection contents and will enable future internal optimizations.

Example usage:
```
var/const/array/foo = array("a","b","c")
foo[1] = "z" // Compile-time error, cannot modify constant collection
```

Note that `var/const/list/L` is not valid in BYOND and usages of constant collections will require a wrapper macro.

## Collection Capacity API

Collections (other than the existing BYOND `list()` and the fixed-length `array()` types) will expose the C# `Capacity` property and `EnsureCapacity()` functionality. These operate similarly to `list.len`, except capacity refers to the number of elements the collection's internal data structure can hold without allocating more memory. Ergo a list could have 3 elements and a capacity of 5, and no memory allocation will occur until a 6th element is inserted.

Accessing `list.capacity` (and similar for other collection types) will return the currect value of the C# `List.Capacity`. Setting it will set it in C#, which may runtime if the new capacity is smaller than the current length of the collection.

Calling `list.ensure_capacity(size)` (and similar for other collection types) will call the C# method `List.EnsureCapacity(size)`. This method ensures that the capacity of this collection is at least the specified capacity. If the current capacity is less than capacity, it is increased to at least the specified capacity.

## BYOND List Deprecation Pragma

A new `DeprecatedByondList` pragma will be implemented. This emits on any usage of BYOND `list()` collections, and will be disabled by default. This pragma exists to assist with guiding migration to OpenDream collections.

## Considerations & Drawbacks

The primary drawback is that these collections types provide no performance benefit to codebases that do not fully migrate to OpenDream, and maintaining BYOND compability relies heavily on `#ifdef OPENDREAM` wrapper macros. That being said, there is still the opportunity for OpenDream to catch misuse of these collections as part of CI workflows.
