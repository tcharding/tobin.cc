+++
date = "2017-02-02T11:38:39+11:00"
title = "Linux Kernel Selftest Framework"

+++

*Kselftest is an effort to enable a developer-focused unit test
framework in the kernel to ensure the quality of new kernel
releases.*  
    - Shua Khan

Install and run Linux kernel selftests on Ubuntu 16.04

<!--more-->

Presentation [slides](reference:
http://www.elinux.org/images/6/61/Linux_Kernel_Selftest_Framework_-_Quality_Control_for_New_Releases.pdf)
by Shua Khan on self test framework.  
Git [repository](git: https://git.kernel.org/cgit/linux/kernel/git/shuah/linux-kselftest.git) for the project.


## Test machine

Product: Intel(R) Core(TM) i5-4590 CPU @ 3.30GHz  
Description:	Ubuntu 16.04.1 LTS  
Kernel: Linux 4.10.0-rc6+ x86_64 GNU/Linux  

## Build on Ubuntu

The following packages are required

libcap-ng-dev  
libnuma-dev  
libcap-dev  
libpopt-dev  

```
$ KTREE = /path/to/kernel/tree  
$ cd $KTREE/tools/testing/selftests  
$ make  
```

## Run

Some tests require root privileges to run

```
$ sudo make run_tests
```
Your machine may appear to die as the tests start.
Tests run in approx 10 minutes on this machine, your results may vary.

Test targets may be specified using

```
$ sudo make TARGETS='size' run_tests
$ sudo make TARGETS='net networking' run_tests
```
Single or multiple targets are accepted. Target names come from
directory names under tools/testing/selftests.
