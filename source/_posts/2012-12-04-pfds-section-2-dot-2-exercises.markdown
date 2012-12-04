---
layout: post
title: "Exercises from PFDS Section 2.2"
date: 2012-12-04 18:48
comments: true
categories: pfds scala
---
Section 2.2's exercises define some optimizations to the section's unbalanced
tree set implementation.

## Exercise 2.2
The implementation of `member` that Okasaki gives for binary search trees in
section 2.2 performs `2d` comparisons for a tree of depth `d` in the worst
case, when searching for a number that is the farthest to the right on the
tree, and when the right path in the tree is of depth `d` itself. This is
because every call of `member` checks whether `x < y` and, if not, whether `y <
x`. This strategy allows immediate detection of the case in which the value
being searched for is contained in the current node, permitting short-circuit
cases like where the searched-for value is in the root node, at the cost of
requiring double the comparisons when traversing rightward.

Exercise 2.2 proposes a strategy in which the second comparison is delayed
until the bottom of the tree. This removes the `2d` worst-case number of
comparisons, at the cost of raising the minimum number of comparisons to the
number of nodes in the shortest path from the root to a leaf. The implementation
is relatively straightforward:

{% gist 4206836 %}

We introduce a new parameter to `member`, `c: Option[T]`, which is `None` when
there is not yet a candidate for equality, and `Some(d)` when `d` is a candidate
for equality. Then, when we reach a leaf node, we check to see whether our
candidate is equal to the value we're searching for.

## Exercise 2.3
Exercise 2.3 proposes an optimization to `insert`. The implementation given by
Okasaki performs wasteful path copying when the value being inserted is already
in the tree. If the value being inserted is present in a particular node, the
only thing that's done is to short-circuit the operation and return the node
as it exists, which doesn't reverse any of the path copying performed while
traversing the tree down to the node.

Throwing an exception in the case where the element being inserted already
exists avoids the wasteful path copying. Okasaki requires that the function
establish only one exception handler per insert, not per function call, so we
catch the exception in the helper `insert` method of the UnbalancedTreeSet
class.

{% gist 4206895 %}

## Exercise 2.4
Exercise 2.4 is to integrate the improvements to `member` from exercise 2.2
into the improved version of `insert` from 2.3. This is straightforward: we
follow the pattern established by 2.2, threading an equality candidate through
the recursive calls, and checking for equality only at the end. Now `insert`
performs no unnecessary path copying if inserting a pre-existing element, and
performs at most `d + 1` comparisons along the way.

{% gist 4207683 %}

## Exercise 2.5
Exercise 2.5 is about generating balanced binary trees that share as much as
possible. The exercise assumes that all of the values in the trees are the same,
which doesn't make much sense for the set abstraction, but ensures that all
subtrees of the same size are identical.

For the first part, we implement a function `complete` that creates a complete
tree of `d` levels. As a binary tree, it should have `2^d - 1` values inside.
The implementation is a straightforward recursive function. The base case, a
binary tree of 0 levels, is just a leaf node. The recursive case creates an
internal node whose children are both the result of recursing with one less
level.

For the second part, we implement a function `balanced` that creates a tree
with `d` values that is as balanced as possible: every internal node's subtrees
differ in size by at most 1.

I put both of these methods directly on BinaryTree's companion object. I think
this is idiomatic, but it's not really clear to me: there are so many different
ways to do things in Scala.

{% gist 4210143 %}

I wrote a few tests for the tree generation functions in a normal
assertion-based style before remembering the existence of
[ScalaCheck](https://github.com/rickynils/scalacheck) and its obvious
applicability for this kind of thing. I've been itching to use a
QuickCheck-style testing framework for a while. If you haven't seen it before,
you should definitely check it out. It allows you to write tests in a
declarative style, without specifying individual test cases. The framework
will generate random test cases for you, either by using built-in random
generators for simple cases like arbitrary strings or integers, or by using
a user-defined random object generator.

In this case, since I wanted my test cases to vary the number of levels or
nodes in a balanced tree, I was able to just use the built-in random integer
generator. Writing tests in this way is concise and fun. Here's what they ended
up looking like:

{% gist 4210365 %}

## Exercise 2.6
This exercise asks the reader to modify UnbalancedTreeSet so that it represents
a map instead of a set. This is very straightforward, if tedious: we must add
another data member to `BinaryTreeNode` that represents the value of an entry
in the map, change `member` to `lookup` and have it return the value data
member instead of true, and throw an exception instead of false, and finally
change `insert` to `bind`, give it an extra value argument, and set the
`BinaryTreeNode`'s value field to that argument. I didn't implement this.

As usual, the new code's on [GitHub](https://github.com/jordanlewis/PFDScala).
