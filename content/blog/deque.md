+++
date = "2016-11-08T11:14:55+11:00"
title = "Deque Data Structure in Golang"
+++

A deque is type of queue which enables adding and removing items from both
ends. Deque ends have such names as *left*/*right*, *front*/*rear* or, as we
will use here, *front* and *back*. The *add/remove* operations on a deque are
typically called *enqueue* and *dequeue*. For ease of explanation but without loss
of generality, we limit discussion to a deque of integers.

<!--more-->

This gives us the following;
```
type Deque struct {
	// Has unexported fields.
}

func (d *Deque) DequeueB() (int, bool)
func (d *Deque) DequeueF() (int, bool)
func (d *Deque) EnqueueB(x int)
func (d *Deque) EnqueueF(x int)
```

One method of implementing a deque is to use two stacks, one representing the
front of the deque and the other representing the back of the deque. *Enqueueing*
and *dequeueing* then become simply a matter of *pushing* or *popping* to, or from, the
appropriate stack. . We will use a simple stack, supporting the operations push, pop, and
length. Previous blog [post](../stack) on implementing stacks in Golang.
```
type stack []int

// push a new integer onto the stack
func (s *stack) push(x int) {
	*s = append(*s, x)
}

// pop: remove and return top element of stack, return false if stack is empty
func (s *stack) pop() (int, bool) {
	if s.len() == 0 {
		return 0, false
	}

	i := len(*s) - 1
	x := (*s)[i]
	*s = (*s)[:i]

	return x, true
}

func (s *stack) len() int {
	return len(*s)
}
```

By now, the astute reader will be asking what happens when one stack becomes
empty and a further request comes to deque from that end. The solution to this
dilemma presents the only complexity in this implementation of a deque. Each
time an item is added to, or removed from, the deque we balance the two stacks to
maintain the invariant that while there are 2 or more items, each stack holds at
least one third as many items as the other. In code, that is;
```
// balance stacks if needed
func (d *Deque) balance() {
	small, big := order(&d.front, &d.back)
	if small.len() == 0 && big.len() == 1 {
		return
	}
	if 3*d.front.len() < d.back.len() ||
		3*d.back.len() < d.front.len() {
		d.rebalance()
	}
}
```
`order()` simply gives us the stacks back in order of size. Ignoring the
implementation of `rebalance()` and time complexity for now, we have a deque that
maintains two *'balanced'* stacks, one holding items for the front of the deque
and the other holding items for the back of the deque. We can enqueue and
dequeue from both ends.

## balancing the deque

Based on [Open Data Structures](http://opendatastructures.org) [Mor], we
maintain the invariant stated above (neither stack falls below 3x the size of
the other). Before giving the implementation let us discuss the running
time. Clearly if we are going to carry out some sequence of operations on
stacks, adding and/or removing some multiple of N, then we are going to be left
with running time of O(N). This would be disastrous since we call `balance()` on
each *enqueue/dequeue* operation. It turns out though, that while the worst case running
time is O(N), the amortized running time is O(1). Stated another way, for M
enqueue/dequeue operations we will have running time in the order of O(M).

For those familiar with the running time complexity analysis of dynamic
arrays, the above statement will not come as a surprise. For a more complete
analysis see [Mor].

One method of balancing the stacks is by way of a couple of additional temporary
stacks and a sequence of *push* and *pop* operations.
```
// rebalance stacks
func (d *Deque) rebalance() {
	small, big := order(&d.front, &d.back)
	half := (small.len() + big.len()) / 2
	tmpB := &stack{}
	tmpS := &stack{}
	mvN(tmpB, big, half) // store half of big
	mvAll(tmpS, small)   // store small
	mvAll(small, big)    // put bottom half of big onto small
	mvAll(small, tmpS)   // restore small
	mvAll(big, tmpB)     // restore big
}
```
As mentioned previously, `order()` simply gives us the stacks back in order of size, the other helper
functions are;
```
// mvN: move n items from dst and push to src
func mvN(dst, src *stack, n int) {
	for i := 0; i < n; i++ {
		x, _ := src.pop()
		dst.push(x)
	}
}

// mvAll: move all items from src to dst
func mvAll(dst, src *stack) {
	mvN(dst, src, src.len())
}
```
See Github for complete
[source code](https://github.com/tcharding/types/tree/master/deques/twinStacks) and tests.

---
#### Bibliography:
[Mor] - **Open Data Structures**, Pat Morin, Edition 0.1  




