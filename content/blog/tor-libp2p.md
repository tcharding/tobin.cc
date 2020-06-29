---
title: "Anonymous peer to peer applicaitons in Rust (rust-libp2p over Tor)"
date: 2020-06-26T14:07:12+10:00
draft: false
---

We can write peer to peer applications in Rust using `rust-libp2p`. We
can anonymize TCP streams by running them through the Tor network. How
about writing anonymous peer to peer applications in Rust that run
over the Tor network?

<!--more-->

Recently I was tasked with investigating whether it was possible (or
practical) to run libp2p traffic over the Tor network. I work for a
company [1] that writes software in Rust that utilises the
[rust-libp2p](https://github.com/libp2p/rust-libp2p) library for its
network layer. We were interested in seeing if we could get our
traffic to run over the Tor network. This post explains how we did it.

Upfront, there is very little that is *new* here. I just 'stood on the
shoulders of giants' so to speak and wired together a few nice open
source libraries. This took me a while to work out though, so in the
name of saving the next guy some time and effort here goes ...

Disclaimer: I started this project knowing basically zero about Tor.
I'm a journeyman, if I have made a mistake please help me out by
telling me. PRs welcome to all repositories mentioned (that I control
:).

I tackled this challenge in three steps and I will explain the
solution using these same three steps:

1. Write a basic libp2p application to use as a proof of concept.
2. Write an application in Rust that can connect TCP streams over Tor.
3. Wire what we learn in (2) into the application we wrote in (1).
4. As they say ... profit!

## Basic networked application in Rust using libp2p

In order to quickly hack up a basic application in Rust that uses
`rust-libp2p` for the networking layer I had to look no further than
the
[examples](https://github.com/libp2p/rust-libp2p/tree/master/examples)
directory of the `rust-libp2p` project. There you will find a simple
ping listener/dialer (peer 2 peer parlance for client/server). Using
that code I hacked up an application by simply separating the listener
and dialer logic into separate functions and slapping a
[structopt](https://github.com/TeXitoi/structopt) CLI on top.

So far so good.

## Arbitrary TCP/IP streams over Tor

Next I had to learn how to interface with Tor using Rust. A few folks
in other languages were doing this and after a bit of digging I
settled on using the [Tor Control
Protocol](https://gitweb.torproject.org/torspec.git/tree/control-spec.txt)
via the very nice [torut](https://github.com/teawithsand/torut)
library. Once again, the examples directory was my friend.

What with not knowing how Tor works and not knowing how the Tor
Control Protocol (TorCP) works I found this pretty difficult, the
fault for this is all my own and I learned a bunch about how I learn
(or don't learn) things in the process. The time I spent doing this
step was the motivation for writing this post.

The code discussed below is in this repository:
[github.com/tcharding/simple-tor-tc](https://github.com/tcharding/simple-tor-tc)

I am running Ubuntu 18 LTS, I installed Tor using the package manager
(`apt`). At first I was starting Tor using systemd but I quickly found
out that, for reasons I still do not full understand, TorCP only lets
you connect to Tor and get the information required for making an
authenticated connection once. You need to spin up a new instance of
Tor each time you want to connect. `torut` handles that for us

```
let child = run_tor("/usr/bin/tor", &mut [
    "--CookieAuthentication", "1",
    "--defaults-torrc", "/usr/share/tor/tor-service-defaults-torrc",
    "-f", "/etc/tor/torrc",
].iter()).expect("Starting tor filed");
let _child = AutoKillChild::new(child);
println!("Tor instance started");
```

This is the default invocation used by systemd on Ubuntu, I saw no
reason to change it. This implies that the default configuration for
Tor works too - win!

Next lets spawn an echo listener (server). I'm omitting the code for
brevity but you can find it in
[echo.rs](https://github.com/tcharding/simple-tor-tc/blob/master/src/echo.rs).

The plan is to create an onion service (previously called a hidden
service) that redirects to the locally running echo listener. This is
all done using the TorCP and `torut`. Lets get a TCP connection to the
TorCP port
```
let stream = let sock = TcpStream::connect(*TOR_CP_ADDR).await?;
```

Using a static global for the address:

```
pub static ref TOR_PROXY_ADDR: SocketAddr = SocketAddr::V4(SocketAddrV4::new(Ipv4Addr::LOCALHOST, 9050));
```

*Note: All the libraries used depend on Tokio, and therefore use the
Tokio types (`TcpStream`, `TcpListener` etc.).*

Now the fun part, as is required by the TorCP spec we connect to the
Tor instance and ask it for the information required to authenticate.
This includes authentication method (we will use cookies), the
location of the cookie file etc.


```
mut utc = UnauthenticatedConn::new(stream);

let info = match utc.load_protocol_info().await {
    Ok(info) => info,
    Err(_) => bail!("failed to load protocol info from Tor")
};
let ad = info.make_auth_data()?.expect("failed to make auth data");
```

Using the information returned by Tor we can authenticate (assuming we
have read access to the cookie file). `torut` wraps all this up nicely
for us (did I mention that I really like this library):

```
if utc.authenticate(&ad).await.is_err() {
    bail!("failed to authenticate with Tor")
}
let mut ac = utc.into_authenticated().await;

ac.set_async_event_handler(Some(|_| {
    async move { Ok(()) }
}));

ac.take_ownership().await.unwrap();
```

I'm not totally across the event handler but from what I understand
its basically the code that runs for async events from Tor, since we
do not need any async events we just ignore this as is done in the
example code. (I actually do not know when Tor produces these events
or what exactly these events are for.)

Onwards and upwards.

We have an authenticated connection to the locally running Tor
instance that we started. We can now use that connection to create an
onion service.

An onion service is made up of the hash of a public key with a 2 byte
checksum and a single byte version number. We need a private key to
use and `torut` will handle generating this and everything to do with
adding the onion service to the Tor network. (See
[here](https://2019.www.torproject.org/docs/onion-services.html.en)
for an explanation of how the onion service protocol works.)

Lets generate a private key to use

```
let key = TorSecretKeyV3::generate();
```

I just realised I never seed any randomness directly, I did not verify
how `torut ` handles this.

`torut` can then command the local Tor instance to create the onion
service for us, this service remains available as long as the TCP
connection to the Tor instance remains open. This explains why the
code has an enormous single function and no smaller functions since
any calls to `Drop` close the TCP connection. (That and laziness on my
behalf for not working out the types required to correctly pass all
this stuff around.)

```
ac.add_onion_v3(&key, false, false, false, None, &mut [
    (PORT, SocketAddr::new(IpAddr::from(Ipv4Addr::new(127,0,0,1)), PORT)),
    ].iter()).await.unwrap();

let onion_addr = key.public().get_onion_address();
let onion = format!("{}:{}", onion_addr, PORT);
println!("onion service: {}", onion);
```

Ok, so now we have a local echo service listening on a local port and
a onion service available via the Tor network that redirects to this
local echo service. Nice.

Next lets connect to the echo service, thus proving that we can make
arbitrary TCP connections over Tor in Rust code.

Getting the `TcpStream` is wrapped in a helper function, here it is:

```
pub async fn connect_tor_socks_proxy<'a>(proxy: SocketAddr, dest: impl IntoTargetAddr<'a>) -> Result<TcpStream> {
    let sock = Socks5Stream::connect(proxy, dest).await?;
    Ok(sock.into_inner())
}
```

`into_inner()` gives us a raw `TcpStream` that we can read from and
write to. `IntoTargetAddr` is implemented on a string that 'looks
like' an onion address i.e., not a multiaddr but something like:

```
vww6ybal4bd7szmgncyruucpgfkqahzddi37ktceo3ah7ngmcopnpyyd.onion:1234
```

Now we have a Tokio `TcpStream` that we can read from and write to as
we wish:

```
stream.write_all(b"ping\n").await?;

let mut buf = [0u8; 128];
let n = stream.read(&mut buf).await?;
println!("received {} bytes: {}", n, std::str::from_utf8(&buf[0..n]).unwrap());
```

BOOM! Arbitrary data sent using TCP via the Tor network.

## rust-libp2p connection over Tor

Now for the holy grail, connect our POC ping application that we wrote
in step one using `rust-libp2p` over the Tor network.

Functionally the listener side of the application does not need to
change. In order to prove this works though, and for convenience, we
start the Tor node when we run the ping listener. This could have been
done differently but it is the simplest way that proves what we want
to prove.

From here on we will be discussing code in the
[github.com/tcharding/ping-pong](https://github.com/tcharding/ping-pong)
repository. This code, as stated above, is based on the ping example
code in the [rust-lib2p](https://github.com/libp2p/rust-libp2p) repository.

For ease of development we hard code the local address used for the
listener, localhost on port 7777. The application accepts a
[Multiaddr](https://docs.libp2p.io/concepts/addressing/) (a
libp2p specified format for addresses) which is the multiaddr for the
onion service we wish to connect to.

I'm omitting the code for creating the onion service because it is
identical to that discussed above, the only addition is converting the
private key into a multaddr. Currently the `torut` and `libp2p`
libraries do not play nicely together. `torut` uses a 34 byte array
for its underlying onion v3 address while libp2p uses 35 (the final
byte being the version number, in this case '3'). Part of the
upstreaming work from this experiment is to resolve this, hopefully by
the time you read this its already upstream. So, there is a bit of
munging to convert the private key we generate into an onion address, add
the port number and feed this into libp2p - all in a days work. We
print the onion address, in multiaddr format, to standard out for the
user to use when starting the dialer.

The dialer is where the real fun starts. In a libp2p application we do
not control anything about dialing except for passing in the multiaddr
to dial. (Well not entirely true, we control the Transport used by the
way we build the libp2p switch (previously swarm), discussion of this
is beyond the scope of this post. See the [libp2p
docs](https://docs.libp2p.io/) for more). So we need to do some libp2p
hacking. To get this working I copied the TCP Transport from the
`rust-libp2p` repo off the master branch. Next some minor code changes
to remove the `async-std` stuff and just use `Tokio`. Then all that was
left to do was hack the dial method to do what we wanted it to do -
connect to the Tor socks5 proxy and return this `TcpStream` instead of
a direct connection. For reference, here is what it looks like before
we started

```
    fn dial(self, addr: Multiaddr) -> Result<Self::Dial, TransportError<Self::Error>> {
        let socket_addr =
            if let Ok(socket_addr) = multiaddr_to_socketaddr(&addr) {
                if socket_addr.port() == 0 || socket_addr.ip().is_unspecified() {
                    debug!("Instantly refusing dialing {}, as it is invalid", addr);
                    return Err(TransportError::Other(io::ErrorKind::ConnectionRefused.into()))
                }
                socket_addr
            } else {
                return Err(TransportError::MultiaddrNotSupported(addr))
            };

        debug!("Dialing {}", addr);

        async fn do_dial(cfg: $tcp_config, socket_addr: SocketAddr) -> Result<$tcp_trans_stream, io::Error> {
            let stream = <$tcp_stream>::connect(&socket_addr).await?;
            $apply_config(&cfg, &stream)?;
            Ok($tcp_trans_stream { inner: stream })
        }

        Ok(Box::pin(do_dial(self, socket_addr)))
    }
```

Patched, it looks like this:

```
    fn dial(self, addr: Multiaddr) -> Result<Self::Dial, TransportError<Self::Error>> {
        let dest = tor_address_format(addr.clone())
            .map_err(|_| TransportError::MultiaddrNotSupported(addr.clone()))?;

        debug!("multi: {}", addr);
        debug!("onion: {}", dest);

        async fn do_dial(
            cfg: TokioTcpConfig,
            dest: String,
        ) -> Result<TokioTcpTransStream, io::Error> {
            info!("connecting to Tor proxy ...");
            let stream = crate::connect_tor_socks_proxy(dest)
                .await
                .map_err(|e| io::Error::new(io::ErrorKind::ConnectionRefused, e))?;
            info!("connection established");

            apply_config(&cfg, &stream)?;

            Ok(TokioTcpTransStream { inner: stream })
        }

        Ok(Box::pin(do_dial(self, dest)))
    }
```

Connecting to the Tor socks5 proxy is done as we do above using the
`tokio-socks` library, here are the relevant lines of code again:
```
let sock = Socks5Stream::connect(*TOR_PROXY_ADDR, dest).await?;
Ok(sock.into_inner())
```

The default TorCP port is hard coded out of lazyness, I'm thinking this
will go in the Transport config so they can be set by the user.

In hindsight pretty simple huh!? I hope you enjoyed this as much as I did.

Happy Hacking -- Tobin C. Harding.


## TODO / upstream work

- Patch `torut` to expose getting the raw onion address bytes
- Patch `rust-libp2p`'s onion multiaddr to add functionality to
  stringify it in a format that Tor understands.
- Try to upstream the Tor Transport to `rust-libp2p` as explained above

## Thank-you's

Thanks to the `torut` authors for saving me from having to interface
with the Tor Control Protocol manually. Thanks to `rust-libp2p` for
being so modular and making it trivial to extend the Tokio TCP
Transport to support connection via Tor. Finally, thanks to my
employer for paying me to work on this.

### reference:

[1] CoBloX blockchain research lab I currently work for: [https://coblox.tech/](https://coblox.tech/)

- The code base we hope to integrate this work into: https://github.com/comit-network/comit-rs

Libraries used/mentioned above:

- rust-libp2p: https://github.com/libp2p/rust-libp2p
- torut: https://github.com/teawithsand/torut
- tokio-socks: https://github.com/sticnarf/tokio-socks

Code I wrote, all code is open source, free as in freedom, and licensed under the GPL v3.

- Simple Tor Control Protocol application: https://github.com/tcharding/simple-tor-tc
- Ping written in Rust using `rust-libp2p` over Tor:  https://github.com/tcharding/ping-pong
