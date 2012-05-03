---
layout: post
title: "PFDS Section 2.2"
date: 2012-05-01 16:33
comments: true
categories: pfds scala
---
Section 2.2 presents immutable sets implemented with unbalanced binary search
trees, a slightly more complex example of immutable data sharing than the list
example in [Section 2.1]({{root_url}}/blog/2012/01/27/chapter-1-dot-1/). My
first challenge was to reimplement Okasaki's base implementation of unbalanced
binary search tree sets using idiomatic Scala. I had to learn a fair amount
more about Scala's type system to be able to write such an implementation, so I
figured I'd write up some of the things I learned about Scala in the process as
well as the implementation.

## A generic trait for sets
Okasaki's set interface contains two methods, `insert` and `member`. Similar to
the generic stack trait implementation that I wrote about
[last time]({{root_url}}/blog/2012/01/31/abstract-generic-collections-section-2-dot-1-redux/)
, it's easy to create this interface as a generic trait in Scala.

{% gist 2565386 %}

## Unbalanced binary search trees
Unbalanced binary search trees are a great choice for a simple implementation
of Okasaki's set interface, since `insert` and `member` in an unbalanced binary
search tree both take on average `O(log n)` time in the number of elements in
the tree, or `O(n)` time in the pathological case where each subsequent insert
is greater than the last.

I decided to follow Okasaki's functor-like pattern for the implementation, so
I wrote a simple binary tree data type using case classes. This allowed me to
use Scala's pattern matching feature in my implementation of the binary search
tree.

{% gist 2573618 %}

Binary search trees rely on the ordering of the type of element that they're
storing, so a generic binary search tree implementation similarly must require
that its type parameter has an order. In Scala, this can be accomplished with
either of two methods: either by using a *view bound* on the type parameter to
require that it be implicitly convertable to an Ordered type, or by requiring
an instance of Ordering, parameterized on the binary tree's input type, as a
value parameter.

In trying to write the most Scala-idiomatic version of the data structure, I
ended up investigating both methods before deciding on one, since it wasn't
obvious at first which method would be the simplest. If you're already familiar
with Scala view bounds and type orderings, or if you just want to get to the
implementation of the 2.2 data structures already, then you should skip right
ahead to [the implementation](#implementation).

### View bounds
A view bound is specified by the `<%` operator. Specifying `T <% U` in a type
parameter adds the restriction that the type parameter `T` must be *implicitly
convertible* to `U`, which is true when there exists a method
    implicit def t2u(t: T): U
somewhere in the current scope. In our case, we're interested in view bounding
`T` by `Ordered[T]`, which will give us access to the comparison methods (`<`,
`>`, etc) defined on the `Ordered[T]` type for values of type `T`.

For example, to implement a generic less-than function lt that utilizes the
`<` method on the input type's `Ordered` implementation, we write
    def lt[T <% Ordered[T]](x: T, y: T): Boolean = x < y
Without specifying the view bound on `T`, we would not have access
to the `<` method on `x`, since it is not defined for any arbitrary
type `T`.

### Ordering instances
The other Scala-idiomatic way to provide or access the ordering of a type `T`
is through objects that implement the trait `Ordering[T]`. An object that
implements `Ordering[T]` provides the `compare(x: T, y: T)` method, which acts
as the underlying implementation for the trait's other methods, which include
`lt(x: T, y: T)`, `gt(x: T, y: T)`, and the like.

For example, to implement a generic less-than function lt that utilizes the
`lt` method on the input type's `Ordering`
implementation, we could write the curried method
    def lt[T](x: T, y: T)(implicit val ordering: Ordering[T]): Boolean = ordering.lt(x, y)
Note that we don't have to specify a type bound on `T`, but we still ensure
that `T` is comparable at the type system level by requiring as an argument an
ordering object that's parameterized on `T`. We make the ordering an implicit
and curried argument, so that if `T` has an implicit conversion to
`Ordering[T]`, as most of Scala's comparable types do, the user doesn't have to
explicitly pass in the ordering.

Using instances of `Ordering[T]` makes your code slightly less pretty, as you
can no longer write `x < y`, you have to instead write `ordering.lt(x, y)`, but
it makes more explicit what's going on under the hood. It wouldn't necessarily
be obvious that writing `x < y` where `x` and `y` are instances of type `T <%
Ordered[T]` actually invokes the `<` method on whatever `Ordered[T]` object `x`
and `y` are implicitly convertible to. I think I prefer the more explicit
version using `Ordering[T]`, but that's probably because using implicit
conversions at all leaves me with a funny taste in my mouth. It seems to me
that overusing implicit conversions would lead to a special hell of confusing
spaghetti code.

<a id="implementation"></a>
## Scala implementation of unbalanced binary tree sets
For the implementation, I wrote the actual `insert` and `member` logic as
private methods on a utility object. The class that actually implements the Set
trait uses the utility object as its underlying implementation. I used Scala's
*companion object* idiom to structure the whole thing in an elegant way.

Here's the companion object that contains the underlying implementation logic:

{% gist 2573799 %}

And here's the class that actually implements the Set interface using the
companion object:

{% gist 2573976 %}

You can see the code in its entirety, as well as a small unit test, on
[GitHub](https://github.com/jordanlewis/PFDScala). Next time I'll implement
solutions for section 2.2's exercises.
