+++
date = "2016-10-26T09:54:21+11:00"
title = "Queue Data Structure in Golang"
+++

A queue is a container that supports retrieval by first-in, first-out (FIFO)
order. The *get* and *put* operations for a queue are usually called *enqueue* and
*dequeue*, other operations may include *isEmpty*. A full description of
queues can be found online [here](http://opendatastructures.org).

Queue's minimise the maximum time spent waiting, however the average wait time
will be the same whether a LIFO or a FIFO is used.

<!--more-->

A queue can be easily implemented in Go using slices.

## queue of integers
```
package queue

type Queue []int

func (q *Queue) Enqueue(x int) {
	*q = append(*q, x)
}

func (q *Queue) Dequeue() (int, bool) {
	if q.isEmpty() {
		return 0, false
	}

	x := (*q)[0]
	*q = (*q)[1:]
	return x, true
}

func (q *Queue) isEmpty() bool {
	return len(*q) == 0
}
```

We can then use our queue of integers like this
```
func fn() {
    var q Queue

    q.Enque(1)
    q.Enque(2)
    q.Enque(3)

    x := q.Dequeue()  // x = 1
    x = q.Dequeue()   // x = 2
    x = q.Dequeue()   // x = 3
}
```
## queue of structs

Today I found a nice implementation of a Queue in the
[graph](https://github.com/gonum/graph/blob/master/internal/linear/linear.go)
package by the people at [Gonum](https://github.com/gonum). I have stripped it
down to a queue of structs
```
// Queue implements a FIFO queue.
type Queue struct {
	head int
	data []item
}

// Item: real implementation would have something useful here.
type item struct {
	data string
}
```
`head` is the index of the head of the queue. Length of the queue is then the
length of the data slice less the index of the head item.
```
// Len returns the number of items in the queue.
func (q *Queue) Len() int { return len(q.data) - q.head }
```
In order to reuse the underlying array, when an item is enqueued, if there is
room at the front of the queue then we copy the current queue contents back to
the start of the array.
```
// Enqueue adds the node n to the back of the queue.
func (q *Queue) Enqueue(n item) {
	if len(q.data) == cap(q.data) && q.head > 0 {
		ln := q.Len()
		copy(q.data, q.data[q.head:])
		q.head = 0
		q.data = append(q.data[:ln], n)
	} else {
		q.data = append(q.data, n)
	}
}
```
Note that arguments to copy may be overlapping, Go's built in `copy` allows
this. This is a nice efficiency tweak over the simple implementation given at
the start of this post. We now reduce the number of times that a new underlying
array is created.

The memory usage of this implementation is quite different from the simple
queue. Like as is often the case with data structures, different data structures
may perform differently under different workloads, one is not necessarily better
than another.

---
#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
[CLRS09] - **Introduction to Algorithms**, Thomas H.Cormen, Charles E. Leiserson,
Ronald L. Rivest, Clifford Stein  
[Mor] - **Open Data Structures**, Pat Morin, Edition 0.1  
