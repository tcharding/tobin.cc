+++
date = "2017-07-20T08:28:24+10:00"
title = "Getting Started with Linux Kernel Development - Part 2: The Process."

+++

Part 2 of this series outlines a method for starting to learn the
process of Linux kernel development. As stated in part 1, this is but
one method. The aim of this post is to illuminate a pathway starting
at the point when you have had your first patch merged into the
mainline. If you have not had your first patch merged you may like to
read [part 1]({{< relref "blog/kernel-dev-1.md">}}) of this series.

<!-- more -->

## Introduction

Learning Linux kernel development consists of a number of conceptual
pieces. One way of discussing them is;

 1. Understanding *how* the kernel works.
 2. Understanding *what* kernel code does.
 3. Understanding how to *change* kernel code.

The first in this list takes years. The Linux kernel, like any operating
system, is a large complex software system. Understanding complex
systems takes time and effort.

The second is more tangible, in order to be able to understand kernel
code you simply need experience reading and writing kernel
code. Circular argument I know, but let's move onto number three of
our list for the moment and return to two shortly.

In theory, one can get the kernel source code and hack away merrily in
the privacy of ones local machine. That kind of defeats the purpose
though. Open source is about collaboration, we want to learn how to
work on the kernel so we can contribute to, and collaborate with, the
kernel development community. If we want to be able to understand what
kernel code does we need to read [a lot of] code and write 
code. And as we have established, if we are going to write kernel code
we want to be able to contribute the code back to the community. It is
therefore essential that we first become comfortable with the process of
changing the source and having our changes merged back into the
mainline.

Fortunately, and in contrast to popular opinion, the Linux kernel is
extremely open and welcoming to new developers. There is a complete
system in place that provides new developers with a place to
learn the kernel development process. The aim of this blog post is to
shed some light on this system and outline *one* method of using this
system to learn the Linux kernel development process.

## First Patch Series

At this stage, it is assumed you have a development environment set up
that is amenable to kernel development. You have read all the
documentation, within the kernel tree, in `Documentation/process/*`.
You have subscribed to the
[kernel newbies](https://lists.kernelnewbies.org/mailman/listinfo/kernelnewbies)
mailing list and also the
[device driver](http://driverdev.linuxdriverproject.org/mailman/listinfo/driverdev-devel)
development mailing list. You are sorting your incoming email into
mailboxes so that mailing list mail can be viewed separately from
other mail. Finally, you have had a patch merged into Greg
Kroah-Hartman's staging tree. More specifically, into the
staging-testing branch of that tree. If you are not at this stage you
may like to read [part 1]({{< relref "blog/kernel-dev-1.md">}}) of
this series.

Thus far, most of the information in this blog series is accessible
online without too much difficulty. Primarily on the
[kernel newbies](https://kernelnewbies.org/) web site. What follows
was, at the time of writing (or more accurately, six months
ago when I was at this stage) not easily garnered from the web.

We will be working exclusively patching code within
`drivers/staging`. While *trivial* patches to the kernel proper can at
times get through the merge process it is, in my opinion,
disrespectful of the kernel maintainers time to attempt such
changes. Please learn from my mistakes. Pushing cleanup patches into
subsystems outside of `staging` benefits the kernel very little. Doing
so trades your education for a non-trivial
amount of effort on the part of the maintainer. There is a system in
place for you to learn and that system is the `staging` directory (and
the machinery in place around it).

#### On Motivation

Hackers tend to like working on *interesting*
problems. It is of little use outlining a method for learning kernel
development if it is not interesting. Now arises then the question of
motivation. We must at this stage select some code to work on. Within
`drivers/staging` there are a number of drivers. They are at various
stages of development. Some are on their way into the kernel, some may
be on their way out of the kernel and some may be stuck because of
some design issue i.e implementing an old API. Also the drivers may
vary in how complete or how functional they are.

There are [at least] three things that may be considered in selecting a driver to
work on.

 1. Does the driver present easily accessible code modifications.
 2. Does the driver belong to a subsystem of interest.
 3. Is hardware for the driver accessible.

These three questions are personal and you may or may not be able to
answer them. Also, there may be a trade off between them. You may like
to keep them in mind as you navigate your way around staging. You need
not confine yourself to a single driver, however, I found it more
engaging to do so.

#### On Hardware

It is not necessary in order to complete the steps in this blog post
to have access to real hardware. It is however necessary if you
intend to continue down the path of writing a functioning device
driver. Doing so is often stated as a stepping stone to kernel
development. A web search on the driver name will usually turn up some
information on the hardware that the driver is intended to run on. While
you may not like the idea of spending $200 on a piece of hardware that
you don't need if you are going to spend 3 months writing a device
driver then the investment in your education is well worth it. You may
also like to consider the machine that the device requires, for
example if you intend working on a driver for an SDIO device you will
need a machine that has an SD card reader that supports the SDIO
protocol.

#### Getting Started

We will be using `checkpatch.pl` to find some code to work on. If you
are still looking for a driver to work on you may like to run
checkpatch on the whole staging directory and pick a driver that
throws a lot of warnings.

    scripts/checkpatch.pl -f --terse --strict drivers/staging/*/*.c

You may like to set up a shell alias for checkpatch

    alias checkpatch='/path/to/kernel/tree/scripts/checkpatch.pl -f'

You need the `-f` option to have checkpatch work on source files, as
apposed to unified-diff formatted patches.

At this stage we have a driver we are going to work on (henceforth
referred to as *your driver*). We have the output from
checkpatch providing ideas for code to work on. Let us now start with
the actual steps intended for this blog post.

### 1. Pick your Fixes

Using the above alias run checkpatch on the source (or header) files within your driver.

    checkpatch --strict --terse --show-types *.c

Henceforth, for brevity, we will refer to all checkpatch outputs i.e ERROR, WARNING,
CHECK as 'warnings'.

As you did when creating your first patch, have a look at the
different types of warnings (`--show-types`). Read the code around the line in
question. At this stage there is little hope that you will understand
what the whole driver is doing. We are aiming, however, to fix
warnings for which we
do understand the surrounding code. At a minimum you should understand
what the surrounding block of code does. 

Pick three warning types that you feel you understand. For each
warning type fix *all* instances of that warning type within the
driver, both header files and source files. Once you have fixed all
instances of a single warning type commit your changes. You read
[part 1]({{< relref "blog/kernel-dev-1.md">}}) right? You read
`Documentation/process/submitting-patches.rst` right? You read the
section `2) Describe your changes` right? If you want to have any hope
of getting *real* code changes into the kernel you need to be able to
describe your changes. The place to do this is the git log
message. The time to learn how to do this *well* is now. Even though
your initial changes will be very simple you should describe them
fully and thoroughly as outline in the above mention kernel document,
specifically section 2.

At the risk of repeating myself, no one *has* to respond to your
email/patches when doing kernel development. If you want your patches
to be taken seriously you *must* present them in the correct
manner. The more you display that you understand the process and
correctly adhere to it the better your chances of success in kernel
development.

Please use correct, both grammatically and technically, descriptions
of your code changes. Use the correct format for the git log message
and for the summary phrase. The summary phrase will be used in the
subject line when your patch is in email form. This information is
contained in section `14) The canonical patch format` of
`submitting-patches.rst`. Also, every other patch on the device
drivers mailing list *should* have the same form. Newbie mistakes
not-withstanding. There is a strict form for a reason, you will have
better success if you follow it.

### 2. Check your Fixes

At this stage you have three commits in your git history and you have
been working on a branch created off of staging-next. Now we are going
to try to eliminate mistakes.

#### On Making Mistakes in Public

Kernel development is done in the open. I believe that one of the
reasons people are so intimidated by LKML and kernel development in
general is that making mistakes in public hurts. It hurts
bad. There is nothing you can read that is going to make it hurt less
when (and you will) you write some brain dead comment in an email on a
kernel mailing list. Or when you breach some *simple* protocol that
you well know, making you look like a complete goose. Mistakes happen,
mistakes are tolerated. Repeated mistakes, especially after being
corrected, are not so well tolerated. And I personally do not see why
they should be. We are striving for excellence, mediocrity has no
place here. Continually striving for excellence is one of the reasons
open source software is such a beautiful pursuit.

**1. Read the diff for each commit**.

Patches submitted to kernel mailing lists are typically reviewed
by reading the diff. At first this may seem difficult since there are
some peculiarities to the diff format but if you check each of your
patches first by reading the diff you will get good at catching
mistakes. Here is a shell function that you may find useful

        git-log-patch() {
            local commit
    
            if (( $# < 1 )); then
                commit="HEAD~"          # default to last commit
            else
                commit="$1"
            fi
    
            git log --color=always --patch --reverse "${commit}".. | less 
        }


`git-log-patch` accepts as argument the commit prior to that which you wish to
view, as usual this can be a hash or `HEAD~`. It then displays each
commit in succession in patch format.  You may also find this shell
function useful when viewing your git index

    git-log-abbrev () {
        # number of commits from index to view, default 1
        local COMMITS=${1:-20} 

        git log -$COMMITS --pretty=oneline --abbrev-commit --reverse
    }

If you find mistakes in one of the commits then you may like to
rebase in order to fix.

        git rebase -i HEAD~~~

Rebasing takes practice, that is one reason why we are doing a
three patch series. Rebasing a 20 patch series is more prone to
error. You will need to rebase to edit the commit log as well as to
make code changes. It is *very* easy to mix up your commits, changes
from one commit find their way into the next commit if you do not use
the correct git commands when adding files and continuing the rebase.
Any time you rebase I *strongly* suggest re-reading the whole
series. Catching your mistakes yourself is much easier on the ego than
submitting patches with blatant errors. See 'On Making Mistakes in
Public' above.

### 3. Create the Patch Set

At this stage you have three commits in your git index. Each commit
does one thing and one thing only, in this case each commit fixes one
checkpatch warning type. Your git log clearly states the reason for
this patch (checkpatch emits warning XYZ) and clearly states how the
patch goes about fixing the issue. Now we can create the patch series
(or patch set).

    git format-patch -3 -o tmp/path/to/patches --cover-letter

This will output 4 files to the specified path. You can check each
file at this stage for correctness also. We can now turn our attention
to the cover letter. A cover letter is not required but it is not hard
to create and makes the patch set a little more tidy. The content of
the cover letter can range from describing and justifying every patch
to briefly outlining the whole series. For a checkpatch series like
this a quick line or two saying what the series does is
sufficient. The cover letter is also where you may like to indicate
the level of testing carried out. Let's go right ahead and assume you
have not tested your changes. This leads on to the next step in
catching mistakes

#### Verify your Patches Build

Each and every patch that is applied to the Linux kernel must leave
the kernel in a sane state. This means that each of your patches must
**apply and build**. There are a few reasons for this. It aids 
bisecting and bug hunting. Not necessarily every patch in a series
will be applied, a maintainer may pick out a subset of the patches to
apply. And finally down stream [distribution] developers may cherry
pick patches to apply to their kernels.

Patches to staging that do not apply are caught by Greg's robots. You
get an email saying so, no hard feelings. Patches that do not build
however are viewed much more dimly. Again, please learn from my
mistakes. Apply each patch individually and build the kernel for each
patch before submitting a patch series. Here is a link to a
[script](https://github.com/tcharding/kernel/blob/master/apply-and-build.sh)
I wrote to carry out this task.

If you have access to other machines then it is a
nice idea to apply and build your patch set on different
architectures. Particularly Power PC so I read somewhere in the kernel
docs.

Back to the cover letter, you may like to add a line at the end
stating

    Code is untested. Builds on x86_64 and PowerPC.

### 3. Submit the Patch Set

As for your single patch previously the patch recipients can be
garnered from the TODO file within the driver or by running the
`get_maintainer.pl` script on a patch or file. If you manually fill the
`To:` and `Cc:` header fields within the cover letter then you can use
the following git command to send the patch set.

    git send-email --to-cover --cc-cover path/to/dir/*.patch

Once again, you may like to send the series to yourself first to check
it for mistakes.

Now wait. You will get a response from either a reviewer, Greg's
robots, the kbuild test robot, or from Greg himself. If anyone makes
suggestions, thank them for their input and follow their
suggestions. Do not submit a second version (see below) without seeing
to each and every suggestion made by a reviewer.

### 4. Submitting Patch Set Version 2

If changes to your patch set are necessary you will need to submit a
second (and third ...) version. You need to deal with all comments to
your patch on the mailing list. Greg receives *a lot* of patches each
day, and response or contention typically pushes a patch to the bottom
of the list of patches he manages, to get back to the top of the list
(and be merged) you need to incorporate any reviewer suggestions into
a new version. At this early stage you are most likely going to
need to incorporate the suggestion as apposed to arguing your point.

If you get more than one suggestion you can implement all the changes
at once. You do so by rebasing your patch set so as to finish up, as before,
with just three patches. Once you are happy with the new series you must
re-submit it. There are two things to note when doing so. The new
subject for the patch set will be the same as the old except it will
include `[PATCH v2]` in the subject. You can generate this using

    git format-patch -3 --subject-prefix='PATCH v2'

Your cover letter will be the same as for the previous version except
you need to add a section at the bottom that outlines the change
history. Let's assume you are up to version 3, your cover letter may
include something like

    v2 -> v3
    - Fix whitespace issue as suggested by review. 

    v1 -> v2
    - Use u8 instead of uint8.

If you are working on a single patch instead of a series, this change
history is still required. The place to put it is below the `----`
within the patch. Things below the dashed line are not included in the
git index when the patch is applied.

#### On Arguing with Reviewers

TL;DR; Don't argue with reviewers.

Linux kernel maintainers and patch reviewers are in short supply, do
not waste there valuable time arguing with them. At this early stage
it is highly unlikely that you know more about kernel development than
the developer reviewing your patch. Sure, reviewers make mistakes,
sometimes email is misinterpreted. In this case by all means re-state
your position in a different, more thoroughly articulated, manner. If
the reviewer repeats the original sentiment then please accept that
you are probably wrong. There may be something that you have not yet
learned that justifies the reviewers position. A reviewer is not
required to spell out *exactly* why something is wrong, they are
reviewers not tutors. If you ask for pointers as to where you can
learn more about why you are wrong you will do much better than
attempting to prove you are correct. And if finally you still do not
agree with the reviewers position just drop the patch and move onto
something else, you are fixing staging bugs here not re-writing the
scheduler. You are supposed to be learning, forge another patch set on
something different and come back to the contentious issue when you
have learned some more.

The more respectful of other developers time and efforts you are the
more help you will receive. It is not magic, just simple human
interactions.

### Lather, Rinse, Repeat

Above I have attempted to outline a method for learning the Linux
kernel development process while controlling some of the complexity
that is inherent in learning a new complex software system. The issues
touched on are issues I came up against while learning. You may well come
up against different issues, each of us is different with different
skill sets and personalities. One thing is for sure, you will not
learn the process without repeating it many times. So go ahead, enjoy
yourself, fix all the checkpatch warnings in that driver. If there are
not enough there, pick another driver and learn it also. Once you are
comfortable with the process and getting more comfortable with kernel
code you may like to try fixing `Sparse` warnings. Pass the `C=2`
option to your build command to run Sparse.

    make -j9 C=2 M=drivers/staging/FOO

### Final Tips

Here are a couple of final tips you may find useful

* If you finish a patch series late at night or when tired wait until
  the next day before sending it. You may realize you have made a
  mistake once you are better rested.

* Wait at least 24 hours before submitting a new version. This gives
  reviewers more time and allows you to include more changes in each
  version i.e less churn and less demand on maintainers/reviewers.


Good luck
