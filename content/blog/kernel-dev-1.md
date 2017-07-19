+++
date = "2017-07-19T10:28:42+10:00"
title = "Getting Started with Linux Kernel Development - Part 1: First Patch."
+++

This is the first in a series of posts about getting started in Linux
kernel development. Most of what is written in this post is already
available on the web. It is provided here for completeness and as a pre-amble to the next post in the series.

<!--more-->

The aim of this series is to outline *one* way of getting started with
Linux kernel development. It is by no means the only way and it is
quite possibly not even the *best* way.

#### Assumptions

- You are inherently interested in how the Linux kernel works.
- You know the C programming language *well*.
- You know how to use git.

Good luck.


## First Patch

This post will describe steps you may take in order to get your first
patch merged into the mainline kernel.

Do not submit patches to any part of the kernel outside of
drivers/staging whilst getting started with kernel development.

### 1. Set up your tools

#### Editor

The kernel uses a specific coding style, you may like to configure
your editor to support this. 

#### Email

All kernel development is done in the open, this means via mailing
lists. Get comfortable with your email client. Do not use HTML when
sending email to kernel mailing lists.

#### Git

Configure git with your email address and name.

	[user]
		email = joe@email.com
		name = Joe Developer

All kernel patches must contain a tag `Signed-off-by: Joe Developer <joe@email.com>`
  

	

You may wish to add to your git config

    [format]
        signOff = true

**Pro Tip:** If you already have git configured with your GitHub username
then add the real name configuration lines to the git config file
within your kernel development tree (see task 3 below).
                
### 2. Subscribe to mailing lists

- Kernel newbies: kernelnewbies@kernelnewbies.org

- Device drivers: driverdev-devel@linuxdriverproject.org

**Pro Tip:** Do not bother subscribing to LKML at this stage.

You may like to sort emails from mailing lists into
separate mail boxes. Example regex for sorting;

	^List-ID:.*<driverdev-devel.linuxdriverproject.org>$

**Pro Tip:** Always maintain the complete CC list when responding to email
on a kernel mailing list.
        
### 3. Set up development tree

- Clone Greg Kroah-Hartman's tree  

     https://git.kernel.org/pub/scm/linux/kernel/git/gregkh/staging.git/

- Set up tracking branch on staging-next (or staging-testing)


At this stage you may like to limit your scope within the kernel tree to the following;  

    Documentation/process/*  
    drivers/staging/*  
    include/linux/*  
    include/uapi/*  

The reasoning is that it is very easy to get overwhelmed while
learning a new system as large as the Linux kernel. Limiting scope is
one tool useful in controlling the complexity of the task.

### 4. Fix something

1. Run checkpatch against drivers in staging until you find a warning
that you feel you can fix. 

    `$ scripts/checkpatch.pl --terse --strict drivers/staging/foo/*`

    **Pro Tip:** Do not fix line over 80 warnings.

2. Fix *all* instances of the warning within the driver you have
chosen. If a job is worth doing it is worth doing properly.

3. Commit your changes as a single commit, write a *correct* git log
   for the commit. Read `Documentation/process/submitting-patches.rst`
	    
    Git log messages must be of a specific format and content. At
first they are difficult and time consuming to write. You may even
find you spend more time writing git logs at first than code. Keep
at it, you will learn a lot from doing it thoroughly.
 
### 5. Create a patch

At this stage you have created a branch off of staging-next, edited
the source and have a single commit containing the changes. The next
step is to attempt to eliminate mistakes and then create the patch.

1. Check the commit is correct

    `$ git log --color=always --patch --reverse "HEAD~".. | less`

    Read through the diff, this is what reviewers will see. Make sure the
    changes are correct and nothing spurious has snuck into your patch.

    Your commit should build without warnings. During development you may
    like to build the driver (without linking it) using

    `make M=drivers/staging/foo`

    You may also like to build and link the kernel with extra warnings enabled.

    `make -j9 EXTRA-CFLAGS=-W`

2. Output the patch

    `$ git format-patch HEAD~ -o path/to/put/patch`

3. Check you patch applies

    Kernel development moves fast, another patch may have been merged that
    touches the same lines of code. Apply and build your patch on the
    staging-testing branch of Greg Kroah-Hartman's staging tree.

    **Pro Tip:** Work off of staging-next but make sure your patches apply
    to staging-testing before you submit them to the driver dev mailing list.

### 5. Submit the patch

1. Check the TODO file for whom to send the patch to, or use

    `$ scripts/get_maintainer.pl XYZ.patch`

2. Submit the patch. You may like to use git

        $ git send-email
            --to='Greg Kroah-Hartman <gregkh@*****.org>'
            --cc='Joe Maintainer <bar@baz.com>, driverdev-devel@*****.org'
            XYZ.patch

    It is a good idea to first send the patch to yourself to check you
    have everything correct.
    
3. Respond to feedback. If asked to do so, fix your patch and
   re-submit.

    **Pro Tip:** Wait at least two weeks before following up on any email
    sent to a kernel mailing list.

At this stage, if all went successfully, you should get an email from
Greg Kroah-Hartman saying that your patch was merged into
staging-testing. From here your patch will automatically transition to
staging-next then, when the next merge window opens, will by merged into
Linus' mainline.

## Final Note

You should now have a development environment set up suitable for
Linux kernel development. There will of course be further tweaking to
be done, especially in regards to your email setup. You have now seen some the
custom tools used in kernel development, the `scripts` directory
contains many more. You should have learned a thing or two from reading the docs in
`Documentation/process`. Now is probably a good time to read
everything in that directory. Don't worry if it does not all sink in,
you will need to read, and re-read, the docs many times yet.

Getting your first patch merged into the Linux kernel is an exciting
event. Well done if you have got this far, your journey has but
started.

Welcome to the Linux kernel and good luck.

