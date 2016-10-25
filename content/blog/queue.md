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

---
#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
[Cor09] - **Introduction to Algorithms**, Thomas H. Cormen, Charles E. Leiserson,
Ronald L. Rivest, Clifford Stein  
[Mor] - **Open Data Structures**, Pat Morin, Edition 0.1  
