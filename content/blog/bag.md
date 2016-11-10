+++
date = "2016-10-26T10:49:04+11:00"
title = "Bag Data Structure in Golang"
+++

A bag is a container of non-unique items. Bags are defined by the following
operations *Length*, *Add*, *Delete* and *Find*. Bags often also need to support
*FindAll*.

Bags can be ordered or un-ordered. This post will be discussing un-ordered
bags.

<!--more-->

Bags are useful when one needs to store a bunch of things and later check
if a certain thing is present. For example, storing the characters of a string
(and perhaps, the frequency count).

A bag can be easily implemented in Go using a map.

## bag of integers

For the sake of simplicity, we will first cover a bag of integers, then move
onto a bag of characters (including some useful functions on such a bag).

```
package bag

type count int

type Bag map[int]count

func (b *Bag) Len() int {
	sum := 0
	for _, v := range *b {
		sum += int(v)
	}
	return sum
}

func (b *Bag) Add(x int) {
	(*b)[x]++
}

func (b *Bag) Delete(x int) {
	_, ok := (*b)[x]
	if ok {
		(*b)[x]--
	}
}

func (b *Bag) Find(x int) (int, bool) {
	count, ok := (*b)[x]
	return int(count), ok
}

func (b *Bag) FindAll(x int) (int, bool) {
	return b.Find(x) // not useful for this implementation
}
```
We can then use our bag of integers like this

```
func fn() {
    bag := make(Bag)

	// add some values to bag
	for i := 0; i < 5; i++ {
		bag.Add(i)
    }

    // check if we have a '2'
    x, ok := bag.Find(2)  // x = 1 (one '2' in bag), ok = true

    // check if we have a '10'
    x, ok = bag.Find(10)  // ok = false (no '10' in bag)

    // add a second '2' and check we now have two of them in bag 
    bag.Add(2)
    x, ok := bag.Find(2)  // x = 2 (two 2's in bag), ok = true
}
```    

## bag of bytes
A bag of bytes can be used to store such things as byte values for characters
encoded using ASCII. This bag would not be very useful for storing text written
in Greek but would be useful for some cryptography tasks.

The basic operations are similar to the above, except of course we replace
occurences of `int` with `byte`. Also we add a helper function to create a bag
from a string (ASCII values only remember).
```
type Bag map[byte]int

func makeBag(s string) Bag {
	bag := make(Bag)
	for i := 0; i < len(s); i++ {
		bag.Add(s[i])
	}
	return bag
}
```

We can then write some other functions to operate on this bag
```
func (b *Bag) difference(c Bag) Bag {
	bag := make(Bag)
	for k, vb := range *b {
		vc, ok := c[k]
		if ok {
			if vb > vc {
				bag[k] = vb - vc
			}
		} else {
			bag[k] = vb
		}
	}
	return bag
}
```
Similarly, one can implement union, and intersection in the above manner. See
Github for complete
[source code](https://github.com/tcharding/types/tree/master/bags) and tests.

---
#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
[CLRS09] - **Introduction to Algorithms**, Thomas H.Cormen, Charles E. Leiserson,
Ronald L. Rivest, Clifford Stein  
[Mor] - **Open Data Structures**, Pat Morin, Edition 0.1  

