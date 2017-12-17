+++
date = "2017-12-05T16:01:57+11:00"
title = "Software Apprenticeship"

+++

In 2015 I had the very fortunate opportunity to invest some time in my skill as
a programmer. At this time I had an undergraduate degree (Bachelor of Science)
double majoring in software and computer networks. I had also completed the
first two thirds of a masters degree, also in computer science. I chose not to
complete the thesis because I felt I did not have anything to add to the field,
graduating with a diploma of Computer Science (pass with high distinction). It
was during this decision that I became acutely aware of my lack of programming
ability.

I picked up Kernighan and Ritchie [KnR12] and started from scratch. Herein
started a two year apprenticeship of software development.

Why an 'apprenticeship'? I did not start out with much of an idea of what I was
trying to achieve. I just knew that if I was going to get any good at this
programming business I needed more experience. My preferred style of learning is
book reading, and I had done plenty of this at uni. But it was also self evident
that you don't learn to code by reading books. Well why not _actually_ do the
exercises. Then, as long as I picked up the right books I would be assured
success, right?

Over the next two years I would read the best part of 53 computer science text
books and complete the majority of the exercises in any that had them. During
this time I read a number of books and blog posts about software
craftsmanship. Say what you will about the movement, the idea of an
apprenticeship struck a chord with me and I started to view what I was doing as
such. Text book authors were my masters and I was there to learn.

This post is intended for three people.

1) Software developers wishing to understand the path I chose.  
2) Potential employers, wishing to understand my experience.  
3) Family and friends, wishing to know what I was up to all that time locked
away.  

There are many ways to achieve ones goals, mine is but one. Each individual is
different, we learn differently and we engage in different ways. I do not, in
any way, suggest that this is the best path. It is simply the path I chose.

The Beginning
-------------

As stated above the first text I pick up was KnR. To this day, in my opinion,
this is one of the best technical books ever written. If there is perfection in
printed text then KnR is it. My first obstacle was to be able to
concentrate for long periods of time doing mentally taxing work. At first, if I
remember correctly, I could only concentrate for a few hours each day. It was at
least three months before I could complete a full work day and I still remember
with glee my first 12 hour day.

During this stage I committed to organize my time as if I was a remote
employee. I would endevour to work a _normal_ 40 work week and I would log work
session times and topics. (Though during some of the more obsessive phases I was coding
much more than this.) If you are interested the
[logs](https://github.com/tcharding/work-logs) are all online. At first I
started logging by hand. The logs from the start are incomplete. I believe that
I worked on KnR from May 2015 through July. During this time I also separated
from my wife. Be careful kids, too much coding can effect your social life :)
During this time I also settled on a development environment. For the record the
stack I settled on is:

	Emacs, Zsh, Terminator, Mutt, Firefox, git, and gcc.

I have used in the past Ubuntu, Debian, Slackware, and Arch. I now use Ubuntu.

UNIX Programming
----------------

The week before I started undergrad, in a fit of enthusiasm, I went to the store
and bought a copy of Advance Programming in the UNIX Environment (APUE). This
was the 2nd edition. Prior to starting university, my total computer experience
was a term writing logo in year 9. A month or two writing a text based game in
DOS and two months playing around on the internet (primarily on 'hack this
site'). Oh and there was the time I deleted my Mums C: drive when I was a kid,
suffice to say I didn't clock up any hours in front of the keyboard after
that. To say that APUE was a little advanced for me is an understatement. I
however had a number of attempts to read it over the years with varying amounts
of success.

In August I commenced work on APUE for real (I even bought the new 3rd edition
[SnR13]). In parallel I decided to work on another classic UNIX text, UNIX
Systems Programming (USP) [SFR04]. By this stage I was routinely working 12
hours a day stopping only for food breaks. I did this 4 days a week and played
with my kids the other 3 days. It appears, from the logs, that I also started to
read Knuth during this time. I included all reading in logged time as time
worked.

On September 20 I picked up Learning Perl [RS11]. I clearly remember wanting to
have something else to work on to break up the C systems programming. At this
stage there were some low moments when things got rather arduous (SysV IPC I'm
talking to you). On the 2nd of October I completed APUE [SnR13], every chapter
and every exercise. I didn't make it through the last section of USP [SFR04], I
can't remember why. As soon as those two were finished, and while still working
on Perl stuff (moved onto Intermediate Perl [RS12]), I started UNIX Network
Programming (UNP) [SFR04]. This is anther classic UNIX text, Stevens really is a
heavy weight in the field.

October saw the end of Knuth volume 1 [Kn97v1]. It seems I started volume 2
[Kn97v2], the logs don't look like I finished it all in one go but I did finish
it. When I say finish I mean my eyes passed over all the words, how much of it
went in I cannot say :) Eventually I completed volume 2 and started volume 3, I
never did make it far into volume 3 though. 

Security
--------

My next focus would be security and cryptography. Here I made my first of two
attempts at the security programming challenges (previously called 'Matasano')
now online at [cryptopals](https://www.cryptopals.com/). First attempt was using
Perl. Wednesday the 20th October I completed UNP [SFR04]. I picked up Schneier
[BS15] and got cracking (no pun intended). By mid way through November I came to
the conclusion that Python would be better suited to the security challenges and
would also be a useful tool to have in my kit. I ordered Python Essential
Reference [DB09] and started work on
[Python the Hard Way](https://www.learnpythonthehardway.org/). On the 23rd I
restarted the cryptopals challenges from scratch in Python.

Enjoying the crypto challenges, in December, I started working in parallel on
the [Eudyptula Challenge](http://eudyptula-challenge.org/). I also threw up a
few solutions on mathematical programming challenge site
[Project Euler](https://projecteuler.net/). In December I read Version Control
with Git [LMC12] and Beginning Linux Programming [MnS09].

Open Source Contribution
------------------------

From the look of the logs, January comprised a loss of direction. I read a book
on Web development [RS13], continued cryptopals and made my first attempt at
functional programming with The Little Schemer [FnF96] and The Scheme
Programming Language [KD09]. January also hosted what turned out to be a pivotal
text, Clean Code [RM09] by Uncle Bob. This book had a **massive** effect on me
and drives my software development to this day. If I had to pick one book from
the whole apprenticeship to define as *essential* this would be it. January also
saw the start of my first ever open source contributions. This was to the Open
Bazaar Server project, written in Python.

I spent a bit over a month solid working on Open Bazaar. Development was done
via GitHub and I learned a lot from the experience.

Mid way through February I stopped working on Open Bazaar and commenced learning
Golang via The Go Programming Language [DnK16]. During March I also read up on
my shell of choice zsh in From Bash to Z Shell [KPS05]

It was during the months around this time that I thought it a good idea to touch
up on my mathematics skill. I can't say I was extremely successful (I measure
success by having completed a large portion of a text) but I did attempt
multiple texts from various branches of pure and applied mathematics and even
learned a thing or two. With similar brain developmental ideas in mind I
attempted to learn the violin. I practiced an hour a day for a few months
eventually giving up on the idea along with the math study.

Systems Programming
-------------------

On the 12th of March something drove me to pick up The Linux Programming
Interface (TLPI) by Michael Kerrisk [MK10] and get back into systems programming
in C. I should mention here that an undercurrent driving me since January 2013
was a desire to be a Linux Kernel developer. This desire was born out of
attendance of the conference [LinuxConfAu](https://linux.conf.au/) in
Perth 2013. Since then I had hoped to become a competent enough programmer to
try my hand at Kernel development. More on this later. This explains my choice
to do more C coding even though the Kerrisk text is very similar in content to
APUE, amusingly it was SysV IPC that stopped me in this text also. I completed
every chapter and exercise up to, and including, chapter 44. Kerrisk is an
extremely competent writer and I agree with those who say that the mantle has
now passed from Stevens to Kerrisk when it comes to leaning systems programming
in the *nix environment.

I continue working on TLPI until the end of April. Over the next few months I
would read the following operating systems texts

- Modern Operating Systems [AT14]
- Linux Device Drivers 3rd Edition [CRK05]
- Essential Linux Device Drivers [SV08]
- Linux Kernel Development [RL10]

In May I would add ARM Assembly Language [WH09] to the list of completed texts
(including most, but not all, of the programming exercises).

Functional Programming
----------------------

June 2016 would see the start of my second functional programming block. This
included re-reading The Little Schemer [FnF96] and re-working all the code. I
then started one of the highlights of the apprenticeship The Structure and
Interpretation of Computer Programs [ASS96]. This text needs no introduction,
and for very good reason. I make no excuse when I say that this book was tough
going. The text is comprised of 5 chapters. I worked every exercise in the first
4 chapters. Chapter 5 implements an interpreter but by the end of chapter 4 I
was getting lost. I did not manage to complete chapter 5. During this phase I
used GNU Guile Scheme and got a nice little patch accepted into the project. The
patch was a fix to a known bug and removed an erroneous compiler warning that
was emitted every time the REPL started. The patch was in C and Scheme.

August brought more functional programming with
[Learn You a Haskell For Great Good](http://learnyouahaskell.com/) and Real
World Haskell [SGS09] although I did not complete Real World Haskell. Functional
programming continued into September with work on
[Ninety-Nine Haskell Problems](https://wiki.haskell.org/99_questions) and
[CIS194](http://www.seas.upenn.edu/~cis194/spring13/). It was during CIS194 that
I hit a wall with functional programming and decided to move back to world of
imperative codes.

Algorithms
----------

By the end of September I was writing Golang again, with a second reading of The
Go Programming Language [DnK16] in unison with work on the very entertaining
[HackerRank.com](http://hackerrank.com/). I would spend the next three months doing
algorithms problems on this site, eventually making it to the 98th percentile
(of approx 600 000 coders). During this algorithms block I would read, the at
times daunting, Introduction to Algorithms [CLRS09]. I also made study of The
Algorithm Design Manual [SS12] and the online text
[Open Data Structures](http://opendatastructures.org/).

And on into the Kernel
----------------------

Mid January 2017 was linux.conf.au in Geelong. And, as was now customary,
attendance got me super psyched on Kernel development. On returning from the
conference I dropped work on algorithms and started Kernel programming. This
included re-reading Linux Device Drivers [CRK05] and making a (short lived)
attempt at writing a kernel from scratch.

January though to mid March saw various Kernel development tasks. I mistakenly
did some, what I later learned were very rude, clean patches into the core
kernel. PowerPC got to feel my newbie wrath with some clean up patches and also
patches into the GNU Assembler for PPC assembly directives. Continued effort was
applied to the Eudyptula challenge also.

At some stage I started to focus exclusively on the KS7010 Wi-Fi SDIO
driver. During this time I started to share what I had learned about Kernel
development with other upcoming newbies via the Kernel Newbies mailing
list. Somewhere here I got a tutorial presentation on Kernel hacking accepted
into Open Source Summit North America. It was these events and my continued work
on the Kernel that lead me to the conclusion that my apprenticeship was at an
end. But like it says in Apprenticeship Patterns [HO10] in was not until much
later that I realized that these were the closing days.

There were numerous other books I read during this time that didn't get a
mention. Honourable mention goes to The Pragmatic Programmer [HnT99]. For a full
reading list see [~/books](http://tobin.cc/reading-list).

If you got this far, thank you for your time. If this post is in anyway useful
to you I am happy. May your days be full of hard, tractable problems.

---
#### Bibliography:

[ASS96] **Structure and Interpretation of Computer Programs**
Harold Abelson, Gerald Jay Sussman, Julie Sussman

[AT14] **Modern Operating Systems**
Andrew S. Tanenbaum

[BS15] **Applied Cryptography**
Bruce Schneier

[CLRS09] **Introduction to Algorithms**
Thomas H. Cormen, Charles C. Leiserson, Ronald L. Rivest, Clifford Stein

[CRK05] **Linux Device Drivers 3rd Edition**
Jonathan Corbet, Alessandro Rubini, Greg Kroah-Hartman

[DB09] **Python Essential Reference**
David M. Beazley

[DnK16] **The Go Programming Language**
Alan A. A. Donovan, Brian W. Kernig

[FnF96] **The Little Schemer**
Daniel P. Friedman, Matthias Felleisen

[HnT99] **The Pragmatic Programmer**
Andrew Hunt, David Thomas

[HO10] **Apprenticeship Patterns** Guidance for the Aspiring Software Craftsman -
David H. Hoover and Adewale Oshineye

[KD09] **The Scheme Programming Language**
R. Kent Dybvig

[Kn97v1] **The Art of Computer Programming (volume 1)** Fundamental Algorithms -
Donald E. Knuth

[Kn97v2] **The Art of Computer Programming (volume 2)** Seminumerical Algorithms -
Donald E. Knuth

[KnR12] **The C Programming Language**
Brian W. Kernighan and Dennis M. Ritchie

[KPS05] **From Bash to Z Shell** Conquering the Command -
Oliver Kiddle, Jerry Peek, Peter Stephenson

[LMC12] **Version Control with Git**
Jon Loeliger, Matthew McCullough

[MK10] **The Linux Programming Interface**
Michael Kerrisk

[MnS04] **Beginning Linux Programming**
Neil Matthew, Richard Stones

[RL10] **Linux Kernel Development**
Robert Love

[RM09] **Clean Code**, A Handbook of Agile Software Craftsmanship -
Robert C. Martin

[RnR03] **UNIX Systems Programming**
Kay A. Robbins, Steven Robbins

[RS11] **Learning Perl**
Randal L. Schwartz, brian d foy, Tom Phoenix

[RS12] **Intermediate Perl**
Randal L. Schwartz, brian d foy, Tom Phoenix

[RS13] **Programming the World Wide**
Rebert W. Sebesta

[SFR04] **UNIX Network Programming**
W. Richard Stevens, Bill Fenner, Andrew M. Rudoff

[SGS09] **Real World Haskell**
Bryan O'Sullivan, John Goerzen and Don Stewart

[SnR13] **Advanced Programming in the UNIX Environment**
W. Richard Stevens, Stephen A. Rago

[SS12] **The Algorithm Design Manual**
Steven S Skiena

[SV08] **Essential Linux Device Drivers**
Sreekrishnan Venkateswaran

[WH09] **ARM Assembly Language**
William Hohl
 