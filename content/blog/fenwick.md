+++
date = "2017-01-07T11:21:35+11:00"
title = "Fenwick Trees in Golang"

+++

A Fenwick tree is a data structure that holds an ordered collection and supports
the operations `sum` and `update`, both in *O(log n)* time.

<!--more-->

In its most basic form the tree stores an array of integers. `Sum` calculates
the cumulative total of the first *n* integers and `update` modifies an element. Theoretically one is not limited to addition of integers when using
Fenwick trees. We can however limit the discussion of Fenwick trees, without
loss of generality, to trees involving integers and summing/updating using
addition*[1]*. 

See this Github 
[repository](https://github.com/tcharding/types/tree/master/trees/fenwick/fenwick-mul)
for an implementation using multiplication.

## Problem Statement

Design a data structure that supports cumulative accumulation of values and also supports
modification of the values.

## Naive Solution

A naive approach to designing such a data structure may choose to store the values in an array
`[1, 2, 3, 4, 5]`. Summation is then achieved by iterating the array and runs in
*O(n)*. Updating a value in the array is done via indexing and quite clearly
runs in *O(1)* or *constant* time.

Perhaps we may choose to optimize for cumulative summation. This may be done by instead of
storing the values, storing the cumulative totals `[1, 3, 6, 10, 15]`.
Summation is now a matter of indexing and runs in *0(1)* time. However in order to update a value we must
traverse the array from the updated value to the end of the array updating each
value as we go. This is, once again *0(n)*.

## Fenwick Tree Solution

The problem that a Fenwick tree solves is the ability to both sum a series and modify
it in less that *O(n)* running time.

According to [Wikipedia](https://en.wikipedia.org/wiki/Fenwick_tree) Paul
Fenwick proposed the data structure in 1994. How Mr Fenwick came up with this
structure I have no idea but in order to do so he made some very interesting
observations. Let us start with a binary search tree with the nodes 
positioned in such a way that an *inorder* tree traversal would give the same
order as the collection we wish to store. Each node has an associated `value`.

The first observation is that we can update a node by setting the nodes `value`
and then *bubble* the new value up the tree updating each internal node's `value`
*iff* that node was reached via it's *right* child.

Doing so maintains a tree that supports the calculation of
cumulative total by using a similar technique, namely; when finding the sum for a
node we start with the nodes `value` then traverse up the tree, adding the internal nodes `value`
as we pass if the node was reached by its *left* child.

Further explanation can be found in this very nice 
[StackExchange](http://cs.stackexchange.com/questions/10538/bit-what-is-the-intuition-behind-a-binary-indexed-tree-and-how-was-it-thought-a)
post. 

The second observation is that if we take the *inorder*  node position as the
node `id` we can store the tree in an array using the `id` as the index. It can
then be observed that by a feat of *bit twiddling* genius involving adding and
removing the least significant bit we can traverse *up* the tree. Not only can
we traverse up the tree, at each node we can ascertain from which child we
arrived. See the above link for a more thorough explanation.

## Fenwick Tree in Go

Without further ado
```
type Fenwick struct {
	tree []int
}

// NewFenwick: Build Fenwick tree to hold n values.
func NewFenwick(n int) *Fenwick {
	fen := &Fenwick{
		tree: make([]int, n),
	}
	return fen
}
```

Here we abstract the data with a `struct`. This prevents inadvertent
modification of the deceivingly subtle data that makes up the tree. Also, in my
opinion, it makes the slice access in later code cleaner since we need not use
pointer indirection to index into the array as is sometimes required `(*ptr)[i]`.

```
// Sum all values upto and including index.
func (fen *Fenwick) Sum(index int) int {
	sum := 0
	index++
	for index > 0 {
		sum += fen.tree[index-1]
		index -= lsb(index)
	}
	return sum
}

// lsb: Least Significant Bit
func lsb(x int) int {
	return x & -x
}
```
I would love to have gone straight from an understanding of this data structure
to the code, however this is not the case. I merely translated the C code from
[Wikipedia](https://en.wikipedia.org/wiki/Fenwick_tree) into Go code. Even then, it took a while for me to implement the
[test](https://github.com/tcharding/types/tree/master/trees/fenwick/fenwick-mul)
cases thoroughly enough that I understood what was going on.

```
// Update using addition index by value.
func (fen *Fenwick) Update(index, value int) {
	index++
	for index <= fen.Size() {
		fen.tree[index-1] += value
		index += lsb(index)
	}
}

// Size: Number of values stored by tree.
func (fen *Fenwick) Size() int {
	return len(fen.tree)
}
```

In conclusion, then Fenwick tree is a very nifty data structure. Hats off to
Paul Fenwick for his incite. Fenwick trees are used, according to Wikipedia, in
[arithmetic coding](https://en.wikipedia.org/wiki/Arithmetic_coding). They also
find use in counting integer inversion in an array.

---
#### Notes:

[1] Fenwick trees are not limited to integers and addition. Any binary operation may
be used. Any data type which implements the binary operation may be stored in the tree.


