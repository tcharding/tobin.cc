+++
date = "2017-01-05T10:08:31+11:00"
title = "Graph Data Structures in Golang"

+++

*... understand just how astonishingly commonplace (and important) graph problems are*  
*they should be part of every working programmer's toolkit.*
    - [Stevey](http://steve-yegge.blogspot.com.au/2008/03/get-that-job-at-google.html)

The graph data structure is of high utility across the field of computer
science. Graph problems come in many shapes and sizes but once modeled can
typically be represented by a limited number of graph data type variants.

<!--more-->

Writing a general purpose graph library is no mean feat. This post is not about creating a
general purpose graph library, if you are after such a library there is a nice
general purpose graph library written in Go by the
[gonum](https://github.com/gonum/graph) team.

This post will lay the foundation for further exploration of graph algorithms in
Go. As such we will discuss the basic structure of various graphs. These details
will allow us to further build on these structures when faced with a particular
problem or when attempting to implement a particular well known graph algorithm.

## Unweighted Graph

One of the most simple graphs is an unweighted graph with `N` nodes labelled
`0..N-1`. 

```
type Graph struct {
	NumNodes int
	Edges    [][]int
}

// NewGraph: Create graph with n nodes.
func NewGraph(n int) *Graph {
	return &Graph{
		NumNodes: n,
		Edges:    make([][]int, n),
	}
}
```

Since the nodes are sequentially numbered we can store all edges in a slice
indexed by the source node. For an edge from `u` to `v` (`u -> v`) we append `v`
to the slice at `Edges[u]`. We then have a slice of slices representing all
edges in the graph, with each slice `Edges[n]` representing all nodes adjacent
to node labelled `n`. 

The graph thus far is *undirected*, an edge `u -> v` does not imply an edge `v
-> u`. If the problem we are modelling requires an undirected graph it is simply a
matter of adding an edge `v -> u` any time we add edge `u -> v`.

```
// AddEdge: Add an edge from u to v.
func (g *Graph) AddEdge(u, v int) {
	g.Edges[u] = append(g.Edges[u], v)

	// For undirected graph add edge from v to u.
	// g.Edges[v] = append(g.Edges[v], u)
}
```

We have the graph data structure implemented and are able to build a graph to
model the problem at hand. Depending on the problem we will need to access the
graph data in various ways. One common operation used, among other things, in
graph traversal is to iterate all nodes adjacent to node `n`. This is easily
achieved with the current implementation by iterating the slice at
`Edges[n]`. To process all edges in the graph one simply uses a nested loop to
iterate over `Edges`.

```
func (g *Graph) adjacentEdgesExample() {
	u := 0 // Example node label.

	fmt.Printf("Printing all edges adjacent to Node: %d\n", u)
	for _, v := range g.Edges[u] {
		// Edge exists from u to v.
		fmt.Printf("Edge: %d -> %d\n", u, v)
	}

	fmt.Println("Printing all edges in graph.")
	for u, adjacent := range g.Edges { // Nodes are labelled 0 to N-1.
		for _, v := range adjacent {
			// Edge exists from u to v.
			fmt.Printf("Edge: %d -> %d\n", u, v)
		}
	}
}
```

## Weighted Graph

The next level of complexity that we may wish to add to a graph is edge
weight. This can be achieved by defining an `Edge` type then storing all `Edge`
instances in a slice in a similar fashion to the unweighted graph above. We
choose to store the `From` node label in the `Edge` type even though it is
redundant (since `Edges` is inedexed by source node label). This often leads to
code that is easier to write and read. There is obviously a space cost to this
design choice.

```
type Graph struct {
	NumNodes int
	Edges    [][]Edge
}

type Edge struct {
	From   int
	To     int
	Weight int
}

// NewGraph: Create graph with n nodes.
func NewGraph(n int) *Graph {
	return &Graph{
		NumNodes: n,
		Edges:    make([][]Edge, n),
	}
}

```

Edges are added to the graph thus

```
// AddEdge: Add an edge from u to v.
func (g *Graph) AddEdge(u, v, w int) {
	g.Edges[u] = append(g.Edges[u], Edge{From: u, To: v, Weight: w})

	// For undirected graph add edge from v to u.
	// g.Edges[v] = append(g.Edges[v], Edge{From: v, To: u, Weight: w})
}
```

And edges may be processed simply using slice iteration.
```
func (g *Graph) adjacentEdgesExample() {
	u := 0 // Example node label.

	fmt.Printf("Printing all edges adjacent to %d\n", u)
	for _, e := range g.Edges[u] {
		fmt.Printf("Edge: %d -> %d (%d)\n", e.From, e.To, e.Weight)
	}

	fmt.Println("Printing all edges in graph.")
	for _, adjacent := range g.Edges {
		for _, e := range adjacent {
			fmt.Printf("Edge: %d -> %d (%d)\n", e.From, e.To, e.Weight)
		}
	}
}
```

## Arbitrary Node Labels

Graph nodes will not always be labelled sequentially. If we require a graph with
arbitrary graph labels we will likely use a `map` instead of a slice to store
edges. This provides flexibility at the cost of efficiency. Although map access
is said to be have constant time cost, I have found that on large data sets use
of maps is noticeably slower than use of slices. If efficiency is critical
you may like to consider remodelling your problem to use sequential node labels
if at all possible. The code using maps is similar to the code above, here we
will show an unweighted graph. Edge weights may be added as they were for
sequential node labels above.

**Herein we use a type definition to abstract away the node label type.** Also we
now refer interchangeably to node label or node id.

```
type ID int

type Graph struct {
	Nodes map[ID]struct{}
	Edges map[ID]map[ID]struct{}
}

// NewGraph: Create graph.
func NewGraph() *Graph {
	return &Graph{
		Nodes: make(map[ID]struct{}),
		Edges: make(map[ID]map[ID]struct{}),
	}
}

// AddNode: Add node id to graph, return true if added (ID's are unique).
func (g *Graph) AddNode(id ID) bool {
	if _, ok := g.Nodes[id]; ok {
		return false
	}
	g.Nodes[id] = struct{}{}
	return true
}
```

Here we add the constraint that node labels are unique. We use the Go idiom of
mapping an integer (node label) to an empty object `struct{}{}`. While making the
code longer this makes explicit our intention that only the map key is of utility.

Edges may be added thus.

```
// AddEdge: Add an edge from u to v.
func (g *Graph) AddEdge(u, v ID) {
	if _, ok := g.Nodes[u]; !ok {
		g.AddNode(u)
	}
	if _, ok := g.Nodes[v]; !ok {
		g.AddNode(v)
	}

	if _, ok := g.Edges[u]; !ok {
		g.Edges[u] = make(map[ID]struct{})
	}
	g.Edges[u][v] = struct{}{}

	// For undirected graph add edge from v to u.
	// if _, ok := g.Edges[v]; !ok {
	// 	g.Edges[v] = make(map[ID]struct{})
	// }
	// g.Edges[v][u] = struct{}{}
}
```

Prior to adding an edge we access the inner map to verify that it is available
and create it if not. Once again we use an empty struct to make explicit that
the key value is the only value of utility. An undirected graph is created by
adding an edge in the reverse direction as previously discussed.

Edge processing proceeds accordingly.

```
func (g *Graph) adjacentEdgesExample() {
	u := ID(0) // example node ID.

	fmt.Printf("Printing all edges adjacent to %d\n", u)
	for v := range g.Edges[u] {
		// Edge exists from u to v.
		fmt.Printf("Edge: %d -> %d\n", u, v)
	}

	fmt.Println("Printing all edges.")
	for u, m := range g.Edges {
		for v := range m {
			// Edge exists from u to v.
			fmt.Printf("Edge: %d -> %d\n", u, v)
		}
	}
}
```

Experienced Go programmers may pause at the use of a single variable `v` above when
iterating a `map`. Note that this refers to edge `v` not the usual idiom `k, v :=
range m`.

## Conclusion

Graphs are an essential data structure. Implementing them in Go is both
enjoyable and educational. We have presented a simple method of implementing
graphs here but with just this much work we have a data structure which allows us to
do traversal, path finding, minimum spanning tree and much more.

---
#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
[CLRS09] - **Introduction to Algorithms**, Thomas H.Cormen, Charles E. Leiserson,
Ronald L. Rivest, Clifford Stein  

