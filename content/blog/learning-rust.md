---
title: "Learning Rust"
date: 2019-07-31T19:18:01+10:00

---

A few years ago I read about a code training method involving writing the same
algorithm out every day.  I liked the idea but never got around to doing it.  A
couple of months ago I got a job working on a code base in a language I had
never used before - Rust.  Not only did I want to come up to speed as quickly as
I could but I wanted to get *good* at Rust **fast**.  Enter algo training.  This
post will outline what I've done for the last two months and reflect upon it.

First I'd like to shout out to all those who wrote
[The Rust Programming Language](https://doc.rust-lang.org/stable/book/).  Its
open source and it is *very* well written.  If you are planning on learning Rust
you should definitely read it.

Rust has kind of a steep learning curve.  It was after I read the Rust book
(above) and started trying to write some code that I realised that the nuances
of Rust were going to be harder to grasp than some other languages I've learned
in the past.  This realisation was what sparked the idea to do algo training.

The idea behind algo training is you pick an algorithm and write it out every
day - simple.  Why would you want to do that?  Well writing the same code allows
you to write without thinking too much about the logic of what you are writing
so you can focus on other things, other things that are good to be good at as a
professional programmer.  These may be different for each individual but in this
case the primary thing I was focused on was learning the syntax of Rust.  If you
haven't written Rust and are laughing right now then go read some Rust code and
you will get what I mean.

This was also a good excuse to practice my test driven development (TDD).
Interestingly I found a bug in one of the algorithms I was writing after writing
(and testing) it for a few weeks.  This highlighted that I was *not* correctly
following TDD and was a very educational bug to find.  This bug was what really
got me excited about this learning method and made me want to share it.  I am,
hands down, a more rigorous developer now because of that bug - WIN.

The algorithms I chose are not really that important as such.  I would pick a
simple algorithm, simple logic but also simple as in including only a few new
syntactical and/or language features.  Then write it out each morning at start
of day.  Code incrementally using TDD.  Once I was super quick at typing it up,
no mistakes, and putting no real thought into it then I'd choose another
algorithm.  Sometimes I'd write a few on one day sometimes just one.  I used
[Rosetta Code](http://rosettacode.org/wiki/Rosetta_Code) for inspiration.  I was
even able to contribute the bug fix I mention above back to Rosetta Code, that
was nice.

For completeness the algorithms I have used thus far, and the order I added them,
are:

1. linear search i.e., loop over an array
2. binary search
3. a vector macro
4. usage of combinators
5. quick sort

The code is up on [GitHub](https://github.com/tcharding/rust-training).  If you
have any suggestions or improvements on this method please email me (address is
on my GitHub page.

thanks



