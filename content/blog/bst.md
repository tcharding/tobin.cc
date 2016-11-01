+++
date = "2016-11-01T11:37:27+11:00"
title = "Binary Search Tree in Golang"
+++

A binary search tree (BST) is a binary tree where each node has a comparable key. As with
any tree, nodes may optionally contain satellite data.

A BST can support many dynamic container operations, including search, minimum,
maximum, predecessor, successor, insert and delete. The defining feature of a
BST is that keys are maintained in an ordered fashion. This makes a BST a useful
data structure for implementing such things as ordered sets and bags.

<!--more-->

BST time complexity at worst case is O(*n*), this occurs when the tree
is a linear chain of *n* nodes. The expected height of a randomly built binary
tree is however *log n*. This leads to the primary attraction of the BST, namely
that many operations have running time complexity of O(*log n*). However, it is worth
keeping in mind the conditions that lead to worst case performance of a binary
tree. Building a tree from a sorted list of keys will produce worst case performance.

The typical data structure used to implement a binary search tree found in the
literature as pseudo-code is
```
Node {
    integer key
    left Node
    right Node
}
```
Translated into Go, with the addition of a parent node pointer to facilitate
more complex functionality, this becomes
```
type Node struct {
    key    int
    left   *Node
    right  *Node
    parent *Node
}
```
In beginning our discussion on binary search trees we will ignore satellite
data. This is justified by the fact that a BST with an integer key alone is
useful for many operations coupled with the fact that the addition of satellite
data involves only minor changes the underlying types.

If one proceeds, using the above node type, to implement typical BST operations,
it will not be long before a subtle problem arises. If one attempts to create an
empty tree by starting with a zero value node `var n Node` the tree will contain
a node with the key value of `0`. This is because the zero value of a node is
equivalent to
```
    var n Node
    n.key = 0
    n.left = nil
    n.right = nil
    n.parent = nil
```
This may not be the desired behaviour. One solution presents itself from
this (overly explicit) snippet. An empty tree can be created using a node pointer instead
of a node `var n *Node`. Another solution, and the one we will use henceforth,
is to wrap a pointer to the root node within another data structure
```
type Tree struct {
    root *Node
}
```
This allows for the zero value of a Tree to be useful `var t Tree`. It also
abstracts the details of an empty tree away from the user.

## Tree Traversal

It is common to visit the nodes of a binary tree in one of three manners,
pre-order, in-order, or post-order. As the names suggest, these involve visiting
the node before, between, or after visiting it's children. For a BST with
integer keys, it is often most useful to utilise in-order tree traversal (tree
walk). Such a tree traversal will give the keys in ascending sorted order.
```
// Walk tree in order calling fn for each node
func (n *Node) inorder(fn func(n *Node)) {
	if n != nil {
		n.left.inorder(fn)
		fn(n)
		n.right.inorder(fn)
	}
}
```
Using this function, and Go's support for closures, we can do things like
```
// Flatten: return inorder slice of keys
func (t *Tree) Flatten() []int {
	var keys []int
	fn := func(n *Node) {
		keys = append(keys, n.key)
	}
	t.root.inorder(fn)
	return keys
}
```

Related to tree traversal are the operations minimum and maximum. By the formal
definition for the relation between the keys of a BST

*The keys in a binary search tree are always sorted. Let x be a node in a binary
search tree. If y is a node in the left subtree of x, then y.key < x.key. If y
is a node in the right subtree of x, then y.key >= x.key* [Cor09]

the node with the maximum key value will be found in the right-most leaf of the
tree
```
// Max: return maximum key from tree
func (t *Tree) Max() (int, error) {
	if t.root == nil {
		return 0, fmt.Errorf("Max() called on empty tree")
	}
	n := t.root.max()
	return n.key, nil
}

// max: find node with maximum key from tree rooted at n
func (n *Node) max() *Node {
	for n.right != nil {
		n = n.right
	}
	return n
}
```
This shows one implication of using a separate Tree type. Methods may be
defined on Node types and/or on Tree types. For consistency Node methods will
return a node pointer giving us access to the node while Tree methods will
return the key value. Your millage may vary depending on the specific usage of
the tree. Minimum can be implemented in a similar fashion, for brevity it is
omitted but full source code can be found
[here](https://github.com/tcharding/types/tree/master/trees/bsts/intKey/unique).

## Insertion and Deletion

This section follows the algorithmic design outlined in *Introduction to
Algorithms* [Cor09]

Insertion into a BST is simply a matter of locating the correct position and
inserting a new node. First we will create a new node with the requisite key
```
// newNode: create a new node from key
func newNode(key int) *Node {
	var n Node
	n.key = key
	return &n
}
```
Then we can insert the node into the tree
```
// Add key to tree
func (t *Tree) Add(key int) {
	n := newNode(key)
	t.insert(n)
}

// insert node n into t
func (t *Tree) insert(new *Node) {
	if t.root == nil {
		t.root = new
		return
	}
	// find position
	var p *Node = nil
	n := t.root
	for n != nil {
	    p = n
	    if new.key < n.key {
		    n = n.left
	    } else {
		    n = n.right
	    }
	}
	// insert node
	if new.key < p.key {
		p.left = new
	} else {
		p.right = new
	}
	new.parent = p
}
```

Thus far we have allowed multiple keys with the same value, this may or may not
be the required behaviour. It is however, trivial to transform one implementation to
the other, requiring only the addition of a check for equality between the new
nodes key and the current key
```
    // ...
    for n != nil {
		p = n
		if new.key == n.key {
			return
		} else if new.key < n.key {
			n = n.left
		} else {
			n = n.right
		}
	}
    // ...

```
Detion is somewhat more involved. One way to think about it is as
follows; if a node has no children it can safely be removed. If a node has only
children in one subtree it can be replaced by that subtree. If a node has two
children it's successor must be found and used to replace the deleted node. Only
the last of these three cases presents any complexity. Firstly, let us get the
successor node to a key value that exists within the tree

```
// Successor: find smallest key value larger than key
// panic if key not present, ok if found
func (t *Tree) Successor(key int) (int, bool) {
	n := t.root.find(key)
	if n == nil {
		panic("Succesor() called with non-existant key")
	}
	next := n.successor()
	if next == nil {
		return 0, false
	}
	return next.key, true
}

// find node by key
func (n *Node) find(key int) *Node {
	for n != nil && key != n.key {
		if key < n.key {
			n = n.left
		} else {
			n = n.right
		}
	}
	return n
}

// successor: return node with smallest key larger than n.key
func (n *Node) successor() *Node {
	if n.right != nil {
		return n.right.min()
	}
	p := n.parent
	for p != nil && n == p.right {
		n = p
		p = p.parent
	}
	return p
}
```
Now returning to deletion. As stated it is not overly complex to remove a node
with zero or one child(ren). Let us now focus on the case of a node (n) with two
children.

This can be split into two cases; firstly if the successor (s) of n is the
direct right child of n (this implies s has a nil left child) then s can be
'transplanted' into the position of n.

If the successor is further down the right subtree of n then it must be further
moved. We can modify the right right subtree of n by removing s and creating a
tree with s as the root and the reaming nodes as the right child of s. Since the
successor is by definition the smallest of n's right children this new tree
maintains correct structure. This tree can then be transplanted into position.

The code to transplant a node is

```
// transplant replaces subtree rooted at u with subtree rooted at v
func (t *Tree) transplant(u, v *Node) {
	if u.parent == nil {
		t.root = v
	} else if u == u.parent.left {
		u.parent.left = v
	} else {
		u.parent.right = v
	}
	if v != nil {
		v.parent = u.parent
	}
}
```

We then wrap the call to Delete (by key) method around a call to delete (by Node),
easing later additions to the Tree type.

```
// Delete node with key from tree
// true if deleted
func (t *Tree) Delete(key int) bool {
	n := t.root.find(key)
	if n == nil {
		return false
	}
	t.delete(n)
	return true
}

func (t *Tree) delete(d *Node) {
	if d.left == nil {
		t.transplant(d, d.right)
	} else if d.right == nil {
		t.transplant(d, d.left)
	} else {
		n := d.right.min()
		if n.parent != d {
			t.transplant(n, n.right)
			n.right = d.right
			n.right.parent = n
		}
		t.transplant(d, n)
		n.left = d.left
		n.left.parent = n
	}
}
```

Often when thinking about algoritms involving binary trees, recursive solutions
come to mind. And while many binary tree operations can be implemented in a
recursive manner some can be re-written in an iterative manner. Some however
cannot (e.g in-order tree walk). We must be careful to understand the
implications of calling recursive functions in a language that does not
implement tail call optimization. It is therefore sometimes prudent to consider an
iterative implementation as well as recursive. Here is one such function

```
// elem: true if node with key exists in tree rooted at node n, recursive version
func (n *Node) elem(key int) bool {
	if n == nil {
		return false
	}
	if key == n.key {
		return true
	} else if key < n.key {
		return n.left.elem(key)
	}
	return n.right.elem(key)
}
```
And an iterative implementation

```
// elem: true if node with key exists in tree rooted at node n, iterative version
func (n *Node) elem(key int) bool {
	found := false
	for {
		if n == nil {
			break // found = false
		}
		if n.key == key {
			found = true
			break
		}
		if key < n.key {
			n = n.left
		} else {
			n = n.right
		}
	}
	return found
}
```

For full source code including additional operations (sum, size, height etc) see
[github](https://github.com/tcharding/types/tree/master/trees/bsts/intKey/unique).

As a final note, binary search trees can take many forms. If you have any
ideas on how I can better make use of search trees, or any comments at all,
please feel free to email me or submit a pull request.

---
#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
[Cor09] - **Introduction to Algorithms**, Thomas H. Cormen, Charles E. Leiserson,
Ronald L. Rivest, Clifford Stein  
[Mor] - **Open Data Structures**, Pat Morin, Edition 0.1  

