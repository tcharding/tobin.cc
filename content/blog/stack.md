+++
date = "2016-10-17T08:28:17+11:00"
title = "Stack Data Structure in Golang"
+++

A stack is a data structure defined by the operations push, pop, peek, and
empty. A full description of stacks can be found in
[this](http://opendatastructures.org) book. A stack of any of Go's basic data
types can be easily implemented in Go using using slices.

<!--more-->

## stack of integers

```
// stack of integers
package stack

type Stack []int

// IsEmpty: check if stack is empty
func (s *Stack) IsEmpty() bool {
	return len(*s) == 0
}

// Push a new integer onto the stack
func (s *Stack) Push(x int) {
	*s = append(*s, x)
}

// Pop: remove and return top element of stack, return false if stack is empty
func (s *Stack) Pop() (int, bool) {
	if s.IsEmpty() {
		return 0, false
	}

	i := len(*s) - 1
	x := (*s)[i]
	*s = (*s)[:i]

	return x, true
}

// Peek: return top element of stack, return false if stack is empty
func (s *Stack) Peek() (int, bool) {
	if s.IsEmpty() {
		return 0, false
	}

	i := len(*s) - 1
	x := (*s)[i]

	return x, true
}
```

We can then use our stack of integers like this
```
func fn() {
    var stack Stack

    stack.Push(1)
    stack.Push(2)
    stack.Push(3)

    x := stack.Peek()  // x == 3
    x = stack.Pop()    // x == 3
    x = stack.Peek()   // x == 2
}
```

It is trivial to replace integers with any other of Go's basic data types
([string](https://github.com/tcharding/types/tree/master/stacks/string), float64, etc). 

## stack of anything

One downside to the above method is that a new stack needs to be written for
each data type. An alternative is to use the empty interface to allow stacks of
anything.

```
// All types satisfy the empty interface, so we can store anything here.
type Stack []interface{}
```

Then any time we pop or peek at an item from the stack we use a type
assertion.

```
var stack Stack
stack.Push("this")

item := stack.Pop()
fmt.Printf("%s\n", item.(string))
```

This comes with the usual warnings that using the empty type interface reduces
the ability of the compiler to catch type errors, one of the benefits of using a
strongly typed language.

Full source code is available [here](https://github.com/tcharding/types/tree/master/stacks).

Also Douglas Hall has done a nice implementation using linked lists instead of
slices. You can find his gist [here](https://gist.github.com/bemasher/1777766).


