+++
date = "2016-12-06T11:14:39+11:00"
title = "HackerRank: First milestone reached."
+++

*To appreciate programming as an intellectual activity in its own right ... you
must read and write computer programs - many of them.*[ASS96]

For the past nine weeks I have been working on programming questions at
[HackerRank](http://hackerrank.com) completing questions in the 'practice area' i.e I have
not competed in any competitive programming competitions offered by the
site. Today I reached the first milestone I had set, namely, to get a top 1000
ranking (96th percentile) in the algorithms sub domain.

<!--more-->
            

### HackerRank.com

First a brief introduction to HackerRank (1). The site is one of many
competitive programming sites available currently. This is not, in any way, a
post about the merits of one site vs another. As such I do not intend to compare
HackerRank to any of its competitors.

HackerRank is divided into two areas. One is the competitive programming
competitions and the other is the practice arena, an area of the site listing
multiple discrete programming questions divided into domains. This post is
concerning the practice arena. I will not be mentioning the competitions again
and may at times take the liberty of referring to the practice arena as 'the
site' in general without further specification.

The site (I did warn you), is divided into domains. The three 'big' ones are
algorithms, artificial intelligence, and functional programming. Each domain is
separate in terms of score and rank. For each domain a user will have score (of
points for completed questions) and a rank (of all users of that domain). Each
domain is also divided into sub domains, such as sorting, graph theory
etc. Questions are rated easy, medium, hard etc and allotted a maximum
score. Partial solutions garner partial scores and multiple submissions are of
course allowed. Also statistics on success rates of completion are listed for
each question.

### Statistics

I focused on the algorithms domain. I used Golang for all questions. As stated
above my first milestone was to get into the top 1000. During the nine weeks it
took me to do this I was aided by the texts listed in the bibliography
below. The primary aim of completing this milestone was to become a better
programmer and more specifically to become a better programmer in Go.

Without further ado;

150 hours over nine weeks  
140 questions attempted  
12 000 lines of code  

### What I learned

All solutions must be complete i.e no libraries, so data structures and algorithms must be
written from scratch. This means that I learned thoroughly, for example, how to
implement and traverse a graph.

The importance of data structures quickly becomes very clear, most used were; queue, stack, tree
(binary search tree, ordered statistical tree, n-ary tree), graph
(directed/undirected, weighted/unweighted).

All questions read input from standard in and expect output on standard out. For
simplicity I tended to use `fmt.Fscanf`. However on occasion IO
was found to be a bottleneck and I learned that `bufio.Scanner` is faster.

To test each solution I used a trick from Donovan and Kernighan [DK16]. In the
main file we declare two global variables `in` and `out`. Also `main()` is
simply a call to another function;

```
var (
	in  io.Reader = os.Stdin
	out io.Writer = os.Stdout
)

func main() {
	solve()
}

func solve() {
    // ...
}
```

Then in the test file, using [table driven testing](https://dave.cheney.net/2013/06/09/writing-table-driven-tests-in-go), we override `in` and `out` in
order to supply test cases and check the output.

```
func Tes(t *testing.T) {
	var tests = []struct {
		input string
		want  string
	}{
	        {"1 2 3\n", "10\n"},
    		{"5 6 7\n", "11\n"},
	}

	for _, test := range tests {
		out = new(bytes.Buffer)
		in = bytes.NewBufferString(test.input)
		solve()

		if got := out.(*bytes.Buffer).String(); got != test.want {
			t.Errorf("input: %s\n got: %s\n want: %s\n",
				test.input, got, test.want)
		}
    }
}
```

Generally speaking the easy questions are solvable with an O(N^2) algorithm. Once
onto the medium questions however typically an O(NlogN) solution is
required. Often it suffices to look over the solution and change the algorithm
as needed but at times bench marking is helpful. I learned that Go's built in
benchmark facilities are quick and easy to use and provided all one needs to get
the running times down on trickier questions.

Bench marking can be implemented (assuming the question is coded as above) in
Golang by simply adding to a test file the following function;
```
func BenchmarkSolve(b *testing.B) {
	for i := 0; i < b.N; i++ {
		input := "1 2 3\n"
		out = new(bytes.Buffer)
		in = bytes.NewBufferString(input)
		solve()
	}
}
```
A Bench mark file can be produced with `$ go test -run=None -bench=Solve
-cpuprofile=cpu.out`. One can then view the output using `go tool pprof -text
-nodecount=20 ./prog.test cpu.out`. Where `prog.test` is the executable that `go
test` saves by default using the program name and the suffix `.test`. The
`-run=None` flag stops all other tests from being run during the bench mark.

### Conclusion

I found HackerRank to be well written, both the content and the site
operations. The questions were well paced and progressed in difficulty at a nice rate. There
are ample questions and a nice little dopamine release generated
by each score increase to keep you coming back. A text book or two on data
structures and algorithms is surely advisable. I found Golang to be very
suitable and pleasant to work with even though it can be verbose at times. If
you would like to deepen your understanding and quicken your algorithmic output
I can wholeheartedly recommend a few weeks spent on HackerRank.


---
#### Notes:
(1) **I am not affiliated in any way with hackerrank.com**

#### Bibliography:
[Ski08] - **The Algorithm Design Manual**, Steven S. Skiena  
[CLRS09] - **Introduction to Algorithms**, Thomas H.Cormen, Charles E. Leiserson,
Ronald L. Rivest, Clifford Stein  
[Mor] - **Open Data Structures**, Pat Morin, Edition 0.1  
[ASS96] - **Structure and Interpretation of Computer Programs**, Harold Abelson
andGerald Jay Sussman with Julie Sussman.  
[DK16] - **The GO Programming Language**, Alan A. A. Donovan, Brian
W. Kernighan.  
