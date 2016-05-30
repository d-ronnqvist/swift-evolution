# Lazy evaluation when assigning static variables

* Proposal: [SE-NNNN](https://github.com/apple/swift-evolution/blob/master/proposals/NNNN-name.md)
* Author(s): [David Rönnqvist](https://github.com/d-ronnqvist)
* Status: **[Awaiting review](#rationale)**
* Review manager: TBD

## Introduction

Both stored type properties (`static`) and lazy stored properties  (`lazy var`) are lazily initialized. However, they have different initialization behavior in that stored type properties evaluate even when assigning them a new value.

The following code will print `"static"`, but not `"lazy"`:

```swift
class Foo {
    static var bar: String = {
        print("static")
        return "Default"
    }()
    
    lazy var baz: String = {
        print("lazy")
        return "Lazy"
    }()
}

Foo.bar = "Set" // this evaluates the initial value of `bar` before setting a new value

let foo = Foo()
foo.baz = "Set" // this doesn't evaluate `baz` before setting a new value
```

Swift-evolution thread: [[Discussion] Difference between static and lazy variables regarding evaluation of closure](http://thread.gmane.org/gmane.comp.lang.swift.evolution/14086)

## Motivation

Swift currently evaluates stored type properties even when assigning a new value. This behavior is very subtle and can lead to objects being needlessly initialized and "immediately" de-initialized, as well as unwanted side effects (caused by the initialized objects).

For example, a shared re-assignable instance that is replaced during unit test set up will initialize the "real" object before assigning the test replacement. 

## Detailed design

This proposal seeks to unify the lazy evaluation _on assignment_ of stored type properties (`static`) and lazy stored properties  (`lazy var`) so that the value being replaced isn't evaluated (the current behavior of lazy stored properties). 

However, it seeks to keep their respective behaviors and guarantees regarding multithreaded simultaneous access:

From the [The Swift Programming Language (Swift 2.2)](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/Properties.html#//apple_ref/doc/uid/TP40014097-CH14-ID254) regarding lazy stored properties: 

> If a property marked with the `lazy` modifier is accessed by multiple threads simultaneously and the property has not yet been initialized, there is no guarantee that the property will be initialized only once.

and regarding stored type properties:

> Stored type properties are lazily initialized on their first access. They are guaranteed to be initialized only once, even when accessed by multiple threads simultaneously, and they do not need to be marked with the lazy modifier.

No changes to the syntax is proposed. 

This provides a more consistent lazy evaluation behavior, and fixes a (small) source of potential, subtle bugs.

## Impact on existing code

This proposal changes the lazy evaluation of stored type properties when assigning a new value. 

Any code that is relying on this effect would break in subtle ways. This is hard to detect and migrate, but hopefully very rare (and to the best of my knowledge the behavior that code would be relying upon is undocumented).

## Alternatives considered

One alternative is to be consistent with the stored _type_ properties and always evaluate the initial value, even when re-assigning it. However, this version doesn't address the subtle bugs that can arise from this behavior.

Another alternative is to leave the the respective behaviors as is and mention their differences in The Swift Programming Language guide. This might still be the most viable alternative in case the current behavior is a consequence of their respective implementations with regards to multithreaded access.

-------------------------------------------------------------------------------

# Rationale

On [Date], the core team decided to **(TBD)** this proposal.
When the core team makes a decision regarding this proposal,
their rationale for the decision will be written here.
