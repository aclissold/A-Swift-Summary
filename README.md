A Swift Summary
===============

[*The Swift Programming Language*](https://itun.es/us/jEUH0.l) is a great book
filled with incredibly well-written descriptions and clever examples, and I
very highly recommend it.  It's divided into three sections: "A Swift Tour," a
language guide, and the formal grammar. This document falls in-between the tour
and guide—it's a brief description of all the interesting, surprising, and
unexpected aspects of the language that I came across while reading the book.

This isn't a summary of the entire language; the obvious, simple, and expected
aspects of it are not mentioned here.

This summary is divided up into the same sections as the language guide, and
follows its order exactly. Each sub-section falls under one of three
categories:

* :exclamation: Important to remember
* :grey_question: Required further insight
* :bulb: Random thoughts/comments

### Table of Contents
[A Swift Tour](#a-swift-tour)  
[The Basics](#the-basics)  
[Basic Operators](#basic-operators)  
[Strings and Characters](#strings-and-characters)  
[Collection Types](#collection-types)  
[Control Flow](#control-flow)  
[Functions](#functions)  
[Closures](#closures)  
[Enumerations](#enumerations)  
[Classes and Structure](#classes-and-structures)  
[Properties](#properties)  

<br />

A Swift Tour
------------

#### :grey_question: Global Scope

> Code written at global scope is used as the entry point for the
program.

That's awesome! But what happens if you have two source files in a project and
attempt to build & run it in Xcode?

It turns out that if you have
more than one source file, one must be named `main.swift` and will be used as
the entry point. Any other file with top-level code will raise `error:
expressions are not allowed at the top level`.

In fact, you can remove the `@UIApplicationMain` attribute from an iOS app
and create a `main.swift` file with nothing but:

``` swift
UIApplicationMain(C_ARGC, C_ARGV, NSStringFromClass(UIApplication), NSStringFromClass(AppDelegate))
```
and your app will still run!

#### :grey_question: Explicit Enum Values

``` swift
enum Rank: Int {
    case Ace = 1
    case Two, Three, four, Five, Six, Seven, Eight, Nine, Ten
    case Jack, Queen, King
    […]
```

If you were to provide explicit raw values of `1` for the first case and `3`
for the second case, what would the third be?

``` swift
enum Test: Int {
    case A = 1
    case B = 3
    case C // equals 4
}
```
Implicit raw values are always incremented by 1 from that of the value before
it. An error will occur if a value is auto-incremented into one already used.

#### :grey_question: anyCommonElements Experiment

> Modify the `anyCommonElements` function to make a function that returns an
> array of the elements that any two sequences have in common.

This wasn't an immediately obvious solution:

``` swift
func anyCommonElements<
    T, U where T: Sequence, U: Sequence,
    T.GeneratorType.Element: Equatable,
    T.GeneratorType.Element == U.GeneratorType.Element>
    (lhs: T, rhs: U) -> [T.GeneratorType.Element] {
        var commonElements: [T.GeneratorType.Element] = [] // [T.GeneratorType.Element]() doesn't seem to work
            for lhsItem in lhs {
                for rhsItem in rhs {
                    if lhsItem == rhsItem {
                        commonElements.append(lhsItem)
                    }
                }
            }
        return commonElements
}
```

<br />

The Basics
----------

#### :bulb: Keywords as Names

> […] you should avoid using keywords as names unless you have absolutely no
> choice.

When will this ever happen? Should the backticks feature even exist? I guess
\`class\` is a little better than clazz at least.

#### :exclamation: Idiomatic `Int`/`UInt` Usage

> Use `UInt` only when you specifically need an unsigned integer type with the
> same size as the platform's native word swize. If this is not the case, `Int`
> is preferred, evn when the values to be stored are known to be non-negative.

#### :exclamation: Type Inference of Literals

> The rules for combining numeric constants and variables are different from the
> rules for numeric literals. The literal value `3` can be added directly to the
> literal value `0.14159`, because number literals do not have an explicit type
> in and of themselves. Their type is inferred only at the point that they are
> evaluated by the compiler.

#### :grey_question: Common Initialism Conventions

``` swift
let http404Error = (404, "Not Found")
```

Does convention dictate that this variable should instead be named
`HTTP404Error`?
[Here](https://developer.apple.com/library/prerelease/ios/documentation/Cocoa/Reference/Foundation/Classes/NSURLRequest_Class/index.html#//apple_ref/occ/instp/NSURLRequest/URL)'s
an example in the frameworks…

#### :bulb: Optional Binding

Both `if let` and `if var` can be used for optional binding, but the former
seems to be much more prevalent.

<br />

Basic Operators
---------------

#### :grey_question: Remainder Operator Type Inference

``` swift
8 % 2.5   // equals 0.5
```

> In this example, `8` divided by `2.5` equals `3`, with a remainder of `0.5`,
> so the remainder operator returns a `Double` value of `0.5`.

Would the remainder of two `Float`s also return a `Double`?

Nope:

``` swift
let x: Float = 1.5
let y: Float = 1
let z = x % y   // of type Float
```

If you try `Float(1.5) % 1` you'll also get a `Float` because Swift will infer
the `1` literal to be a `Float` in this context. Pretty neat! But if you try
this:

``` swift
let x: Float = 1.5
let y: Double = 1
let z = x % y
```

You'll get the error message `error: 'Float' is not convertible to 'UInt8'`,
which I assume is an unintentionally cryptic way of saying that Swift doesn't
implicitly convert types.

#### :grey_question: Rigorous Closed Range Operator Definition

> The *closed range operator* `(a...b)` defines a range that runs from `a` to
> `b`, and includes the values `a` and `b`.

What happens if `a` and `b` have the same value? It turns out that the loop will
execute once, not twice. So `for i in 1...1` would be a redundantly silly way to
say "do this once."

<br />

Strings and Characters
----------------------

#### :bulb: A Well-Written Chapter!

<br />

Collection Types
----------------

#### :grey_question: Array Type Inference

> Thanks to type inference, you don't need to specify the type to be stored in
> the array when using [the repeated value] initializer, because it can be
> inferred from the default value:
>
> ``` swift
> var anotherThreeDoubles = [Double](count: 3, repeatedValue: 2.5)
> ```

After thinking about it for a while, I believe this to be a mistake. Using a
repeated value of `1` will still yield a `[Double]` since it's explicitly stated
in the initializer. They probably intended to use `Array(count: 3,
repeatedValue: 2.5)`, at which point the above quotation is true. I submitted
this as an issue on their bug reporter.

<br />

Control Flow
------------

#### :bulb: Break in a Loop Statement

> When used inside a loop statement, `break` ends the loop's execution
> immediately.

The following loop has an interesting (although possibly obvious) property:

``` swift
var i: Int
for i = 0; i < 10; ++i {
    // if i == 9 { break }
}
println(i)
```

It executes 10 times either way, but if you uncomment the `break`, it will print
`9` instead of `10`.

<br />

Functions
---------

#### :exclamation: Automatic External Parameter Names

> You can opt out of this behavior by writing an underscore (_) instead of an
> explicit external name when you define the parameter.

#### :exclamation: Variadic Parameter Types

> The values passed to a variadic parameter are made available within the
> function's body as an array of the appropriate type.

<br />

Closures
--------

#### :bulb: A Well-Written Chapter!

<br />

Enumerations
------------

#### :bulb: Associated Values

The ability to associate values with members of an enumeration is a language
feature I've never even heard of before, so that section is definitely worth
reading twice.

<br />

Classes and Structures
----------------------

#### :exclamation: Memberwise Initializers

> All structures have an automatically-generated *memberwise initializer*, which
> you can use to initialize the member properties of new structure instances.

#### :bulb: Identity Operators and Strings

The "identical to" operator (`===`) checks if two variables or constants refer
to the same class instances.

A `String` is passed around by reference behind the
scenes, and two values may even lazily refer to the same reference. How might
`===` behave for two value types with the same value but different underlying
references?

It turns out that there's no way to play around with this, because `===` only
works for types conforming to the `AnyObject` protocol, and `non-class type
'String' cannot conform to class protocol 'AnyObject'`. Interesting to think
about though.

<br />

Properties
----------
