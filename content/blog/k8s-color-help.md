+++
date = "2017-08-29T20:20:12+10:00"
title = "Kubernetes Colorize Help"
+++

Recently I started hacking on the Kubernetes (k8s) project. The code base is in Golang and is super
fun work on. Development is done via [GitHub](https://github.com/kubernetes/) and
[Slack](https://kubernetes.slack.com).

<!--more-->

Kubernetes is a complex project. It is not super big, in terms of line count, approximately 90 000
at the time of writing. The project is moving quite fast, I read today that they are merging about
30 pull requests a day. Development is divided among special interest groups (SIG). New developers
are directed towards SIG/CLI. As you might guess this group takes care of the command line interface
for Kubernetes. K8s clusters are administered via a command line tool called `kubectl` (pronounced:
'kube-c-t-l' or 'kube control').

Fast forward one month. I've been hacking away merrily now for a while on `kubectl`. This has lead
me to some interesting open source libraries. The [cobra](https://github.com/spf13/cobra) library,
brought to you by the creators of [Hugo](https://github.com/gohugoio/hugo) (the static web site
generator that I am using to write this blog), is one of them.

So, my issue, and the reason I am writing this post, was that I was struggling to decipher the
available options output by use of the `--help` flag. Of course, first thing I did was throw a pull
request at Kubernetes with a proposed solution. Turns out there may be a more simple solution (remember 50%
of bug fixes introduce more bugs (Steve McConnel)). Introducing
[generic colorizer](https://github.com/garabik/grc). This awesome tool written by Radovan Garabík
allows you to, as the name suggests, colorize anything.

It is written in Python, you can get install instructions in the repository, or if you are running a
Debian based system just search the package manager for `grc`. You will need a configuration file in
`~/.grc/grc.conf`


While we are at it lets colorize `go test` as well.

```
# Go
\bgo.* test\b
conf.gotest

# Kubernetes
\bkubectl help\b
conf.kubectlhelp
```

Colorization is then achieved via regular expressions. You will need a configuration file for each
entry above. Here is `conf.kubectlhelp`

```
regexp=  \#.*
colour=blue
-
regexp=--.+?(?==)
colour=yellow
-
regexp=^Other Commands
colour=red
-
regexp=^Usage:
color=cyan
-
regexp=^Examples:
color=cyan
-
regexp=^Options:
color=cyan
```

and `conf.gotest`
```
regexp==== RUN .*
colour=blue
-
regexp=--- PASS: .*
colour=green
-
regexp=^PASS$
colour=green
-
regexp=^(ok|\?) .*
colour=magenta
-
regexp=--- FAIL: .*
colour=red
-
regexp=[^\s]+\.go(:\d+)?
colour=cyan
```

If you like shell aliases you may like to define the following

```
alias kch='/usr/bin/grc kubectl help'
alias go='/usr/bin/grc go'
```

voilà `kch` now presents nice colorized output and you can easily discern the options for whichever
Kubernetescommand you are investigating. Bonus points `go test` is colorized too.

thanks
