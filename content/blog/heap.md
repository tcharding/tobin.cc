+++
date = "2016-11-10T08:29:33+11:00"
title = "Heap Data Structure in Golang"
+++

A heap is a data structure that supports the operations *insert* and
*extract*. Heaps typically come in two varieties, *min heap* (for extracting the
minimum value) and *max heap*. A heap is built using a binary tree where each
node is said to *dominate* the nodes below it. The meaning of dominate depends
on the type of heap being implemented, for a *min heap* the key of each node is
*less* than the keys of both of child nodes.

<!--more-->

From here on, without loss of generality, we will talk about a heap of integers
(keys) ignoring satellite data that may or may not be associated with each
integer key. *Node* and *key* will therefore be used interchangeably.

Like a binary search tree a heap can be implemented using a linked data
structure. There is however, a nifty method of implementing a heap using an
array, thereby reducing the memory requirements since there are no pointers to
store. We store the data as an array of keys and use the index of the keys to
implicitly satisfy the role of pointers.

**The root node is stored at index 1 and all operations are 1-indexed**.

For node at index *k* the left child can be found at index *2k* and the
right child at index *2k + 1*. (Quite clearly, checks must be made prior to
access that an index lies within the underlying array).

One additional limitation must be placed on the tree, that it is *complete*, i.e
all levels of the tree are full with the possible exception of the last level,
and if the last level is not full all nodes are as far to the left as
possible. This limitation results in an array with no holes in it.

A heap as just described may be defined in Go as such;
```
type heap struct {
	xs []int
}
```
We wrap the array inside a struct to abstract the implementation and limit
access to the functions that we define. Care must be taken
to remember that the heap is 1-indexed.
```
func (h *heap) Len() int {
	return len(h.xs) - 1 // heap is 1-indexed
}
```
Insertion of a key into a heap can be achieved by adding the key to the end of
the underlying array. Whilst maintaining the structure of the heap this may
violate the *heap property* of the parent of the newly inserted key, namely the
parent of the newly inserted key may not be dominant. The heap property for the
parent node can be restored by swapping the newly inserted child node with the
parent. This may in turn, result in the parent above violating the heap
property. We can continue this *bubbling* of a node to successively higher nodes
until the heap property is restored. When swapping a node with the parent node
we need not consider the sibling node since if a node dominates the parent then
by definition it dominates the other sibling also.
```
// Insert x into the heap
func (h *heap) Insert(x int) {
	(*h).xs = append(h.xs, x)
	h.bubbleUp(len(h.xs) - 1)
}
```
Note that parenthesis are required to dereference the heap pointer in order to save the
return value of `append()`.
```
func (h *heap) bubbleUp(k int) {
	p, ok := parent(k)
	if !ok {
		return // k is root node
	}
	if h.xs[p] > h.xs[k] {
		h.xs[k], h.xs[p] = h.xs[p], h.xs[k]
		h.bubbleUp(p)
	}
}
```
The function terminates when either the node is in place or it has reached the
root position.

We define functions for manipulating indices as described above;
```
// get index of parent of node at index k
func parent(k int) (int, bool) {
	if k == 1 {
		return 0, false
	}
	return k / 2, true
}

// get index of left child of node at index k
func left(k int) int {
	return 2 * k
}

// get index of right child of node at index k
func right(k int) int {
	return 2*k + 1
}
```
We are now ready to extract the key for which our heap was designed (minimum
or maximum). This key is clearly at index 1, extracting this key however, leaves a hole which
must be filled. Swapping the last key of the array into the hole restores the
heap structure but once again may violate the *heap property*. In a similar
fashion to insertion we can restore the heap property by bubbling this node down
the tree until it is either a leaf node or no longer violates the heap
property. In doing this we must consider both child nodes and swap any
non-dominant parent node with the child node that is *most* dominant in order for
the heap property of these three nodes to be maintained.
```
// ExtractMin: get minimum value of heap
// and remove value from heap
func (h *heap) ExtractMin() (int, bool) {
	if h.Len() == 0 {
		return 0, false
	}
	v := h.xs[1]
	h.xs[1] = h.xs[h.Len()]
	(*h).xs = h.xs[:h.Len()]
	h.bubbleDown(1)
	return v, true
}
    
func (h *heap) bubbleDown(k int) {
	min := k
	c := left(k)

	// find index of minimum value (k, k's left child, k's right child)
	for i := 0; i < 2; i++ {
		if (c + i) <= h.Len() {
			if h.xs[min] > h.xs[c+i] {
				min = c + i
			}
		}
	}
	if min != k {
		h.xs[k], h.xs[min] = h.xs[min], h.xs[k]
		h.bubbleDown(min)
	}
}
```
Caution is again needed here since the array indexing and loop conditionals are
unusual because of the 1-indexing.

See Github for complete
[source code](https://github.com/tcharding/types/tree/master/heaps/minInt) and tests.

### Final Note
This implementation is based on the text *The Algorithm Design
Manual* [Ski08]. In this, the author Steven S. Skiena makes an interesting
observation on the construction of a heap. At first glance one may think to
construct a heap by repeated calls to `insert()`. Inserting into a heap (like
any balanced tree) takes O(log *n*)), so heap construction in this manner has
worst case running time of O(*n* log *n*). We can do better, Skiena notes, if we
observe that a *full*, *complete* tree of *n* nodes has *n/2* leaf nodes. These
leaf nodes may be considered as sub-trees that maintain the heap property (since
they have only a single node). If then, we base a heap on any array we need only
*bubble up* *n/2* nodes in order to achieve a heap for which the heap property
holds. This still gives an *upper bound* of O(*n* log *n*), however Skiena goes
on to show that this leads to a *not quite geometric* series that he then
assures us quickly converges to linear. He does however, caution us that this
benefit may not be that useful if the algorithm we plan to use our heap for is
not governed by the construction (i.e heapsort will still run in O(*n* log
*n*)).

---
#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
