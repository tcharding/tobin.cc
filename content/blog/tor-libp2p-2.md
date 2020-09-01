+++
title = "rust-libp2p over Tor - Version 2"
date = 2020-09-01T11:17:01+10:00
draft = false
+++

This is an update of the [initial](../tor-libp2p) blog post on running
[rust-libp2p](https://github.com/libp2p/rust-libp2p) traffic over the
Tor network. In that post we used the
[torut](https://github.com/teawithsand/torut) library to configure the
tor onion service in code. Turns out, for some use cases, there is a
more simple method. This post runs through that method.

<!--more-->

I have created and hosted on `crates.io` a
[crate](https://crates.io/crates/libp2p-tokio-socks5) that implements a
libp2p transport, for this post I'll be referencing code in the
repository backing that crate and also the examples directory of that
repository.

## Basic networked application in Rust using libp2p

The
[examples](https://github.com/comit-network/rust-libp2p-tokio-socks5/tree/master/examples)
directory contains a module called `ping` that implements the ping
server as we did in the first blog post, copying code from the
`rust-libp2p` ping example.

## Configuring Tor via the runcom file

What I would like to discuss is how we configure the onion service
(formally hidden service). In v1 of this post we used torut to do so.
I have since discovered that by simple adding a few lines to the tor
runcom file (e.g. `/etc/tor/torcr`) we can configure the onion service
and simplify considerably the usage of our tor transport. To the
config file we can un-comment/add the following lines:

```
    HiddenServiceDir /var/lib/tor/hidden_service/
    HiddenServicePort 7 127.0.0.1:7777
```

Once tor runs it will store the onion address for the service we just
configured in the hidden service directory given i.e.,
```
cat /var/lib/tor/hidden_service/hostname
7gr3dngwhk74thi4vv6bm3v3bicaxe4apvcemoxo3hadpvsyfifjqnid.onion
```

We pass this address to the swarm's dial method as one would expect to
with any other multi address.

The final piece of magic that we need to provide is a mapping of onion
addresses to local port numbers. This allows the listen side of the
transport to create a TCP socket and bind it to the port that Tor is
expecting. In the above example the local process listens on port 7777
so when we call `listen_on` with the onion address above this onion
address is mapped to port 7777.

BOOM! That's it, the `Socks5TokioTcpConfig` type takes care of
everything else for us and we are away - anonymous peer to peer
applications in Rust using rust-libp2p over Tor!

Thanks for reading,  
Tobin Harding.