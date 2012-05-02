---
layout: post
title: "Abstract Generic Collections: Section 2.1 Redux"
date: 2012-01-31 00:34
comments: true
categories: pfds scala
---
At the end of my last post, I mentioned that I ended up reusing Scala's build-in
List collection to implement the exercises instead of writing a generic abstract
Stack and sample implementations of those. Since then, I've spent some time
learning about how to implement generic collections in Scala. I came up with
a Stack trait, *a la* Okasaki's Stack signature, and three implementations: the
first two are straightforward translations of the SML structures given in the
book, and the third is a more Scala-idiomatic implementation.

On an organizational note, I've also started a
[PFDScala repository](https://github.com/jordanlewis/PFDScala) on GitHub for
this blog series, so I can have a centralized place to put all of the code I
write for the exercises and examples. I'll still put the relevant snippets in
gists so I can embed them here.

A Generic Trait for Stacks
--------------------------
Scala traits are very powerful. They're basically like mixin classes from Ruby,
in that your class can extend from multiple traits without having the troubles
that plague C++-style multiple inheritance.

Abstract traits can be used like Java interfaces. This is just what we need to
write a generic interface for Stacks:

{% gist 1709179 %}

For those of you unfamiliar with Scala, this is similar to a Java interface or
a SML signature: we're defining a type-parameterized trait/interface/signature
with a bunch of method signatures that classes which extend this trait must
implement themselves. What's up with the weird type annotations, though?

In Scala, annotating a type parameter with "+" marks it as a covariant type
parameter. This means that if we have a type C[+T], a type with a single
covariant type parameter, and if type A is a subtype of type B, then type C[A]
is a subtype of C[B]. We can similarly annotate a type parameter with "-" for
the opposite effect.

The ">:" type operator is the supertype relationship: the type on the left of
it has to be a supertype of the type on the right of it.

So why do we need to annotate our types like this here? In Scala, by default,
types are invariant in their type parameters. This means if we had a
List[Integer], and List's type parameter was invariant, then that list would
not be a subtype of a List[Number], even though it seems like it would be fine
because a List of Integers is just a specialized List of Numbers. The same thing
goes for our Stack type, so we mark its type parameter up as covariant.

However, to keep compile-time type checking sound, Scala has to impose some
restrictions on the types of methods in classes with covariant type parameters,
due to the problem of mutability: a mutable array of type T is actually
*contravariant* in its type parameter, because if you had an Array of type
Any, updating one of its cells to be a subtype of Any like String is not always
legal. So, in our Stack example, we can no longer simply say
    def cons(x: T) : Stack[T]
because the x is appearing in a *contravariant* position, meaning that it has
the potential to mutate state in a way that could break type safety.

Luckily, we can get around this problem of contraviariant position by imposing
a bound on the type of cons's parameter: we say
    def cons(x: U >: T) : Stack[U]
to ensure that the input type of cons is a supertype of the stack's type. This
prevents any type safety issues caused by the potential contravariance of the
formal parameter, and allows users to generalize the type of a Stack by consing
a more general type onto it. For example, one could cons a String onto a
Stack[Int] and get out a Stack[Any].


Implementation 1: Using builtin Lists as the backing store
----------------------------------------------------------
This class uses the builtin List implementation to provide the underlying
storage for the Stack. It also uses Scala's *companion object* feature to
provide a static factory method for creating new ListStacks.

{% gist 1709271 %}

Implementation 2: Using a custom List case class as the backing store
---------------------------------------------------------------------
This class uses a custom implementation of List as the backing store. It's the
same idea as the first one, except we use a set of case classes to match on
instead of just wrapping List's functions.

{% gist 1709288 %}

This shows off Scala's case classes a little bit: they're just like datatypes
in SML, except a bit more verbose to specify. The abstract class LIST is like
the name of the datatype, and the case classes that inherit from it are like
the datatype's constructors. Making LIST *sealed* restricts classes from
extending it unless they're in the same file: this allows the compiler to detect
non-exhaustive matches.

Implementation 3: Implementing the Stack functions within case classes
----------------------------------------------------------------------
This implementation is a bit different from the other two: instead of storing
the actual Stack data as a data member inside of a single StackImpl class, this
puts the implementation inside of case classes that extend the implementation
class, which is made abstract.

This strategy seems the most idiomatic of the implementations I wrote. It
defines its own internal datatype like the custom List implementation, without
the mess of having to match on each of the cases of the custom List every time
it's necessary to operate on the data. I think it produces the most compact
and readable code out of all three implementations.

{% gist 1709315 %}

Conclusion
----------
That was a long-winded post for what was just defining a simple custom
collection interface and a few simple implementations. Again, all of this code is available in the
[PFDScala GitHub repo](https://github.com/jordanlewis/PFDScala). Next time
we'll proceed with the next section, 2.2. Hopefully it will be easier to
implement the examples and exercise solutions with the improved understanding
of Scala's type system that I gathered by writing this post.
