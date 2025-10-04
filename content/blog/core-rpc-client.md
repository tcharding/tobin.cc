+++
date = "2025-10-03T10:49:04+11:00"
title = "Why you should write your own Bitcoin Core RPC client"
+++

If you are an application or library developer and you need something
from Bitcoin Core then you may want to use the JSON RPC API that Core
provides. So far so good. You might then wonder to yourself, "seems
pretty easy, surely someone has written a crate (Rust library) already
that does that, let me just grab it". This post is about why that is
a bad idea. 

<!--more-->

## Application Considerations

When an application wants to hit a Core RPC endpoint there are a few
high level things that you may want to consider:

- What versions of Core do we expect to be able to hit?
- What RPC methods do we need to call?
- When calling those methods what, if any, optional arguments do we
  need to use?
  
All three of these can be answered by instead answering one or both of
these question:

1. What data do we need from Core?
2. What effect do I wish to have on the node?


## Technical Considerations

Now lets assume there exists a client library that wraps all this RPC
stuff up for you, there obviously are, and you want to use one. You
may want to make any of the following technical considerations when
assessing the crate:

1. How are you going to hit the endpoint? (JSON RPC is transport agnostic.)
2. How much of that code do you want to write yourself? I.e., how
   tolerant are you to dependencies and transitive dependencies?
3. Is you usecase blocking or async?

## Why I am not going to write and maintain a production client for you

Hopefully already you can see why I, a library maintainer, am totally
unable to answer the technical considerations for you. I put it to you
that I am also unable to answer the application considerations for you
other than 'support all the versions, all the methods, all the
optional arguments'. Which, even if it were possible, would be a ton of
work.

## The solution

You should do what ever you like, this is open source. But I
personally won't be writing a production client for you because I do
not think its a good use of developer time. Instead I propose this:

If we had a crate that provided types (structs and enums) for _all_
the JSON data returned by the whole Core RPC API for _all_ the
versions then a client writer would need only decide what methods they
want to use, what data they need from those methods, what versions of
Core provide that data and BOOM we are off to the races.

We have such a crate: [`corepc-types`](https://crates.io/crates/corepc-types)

One small caveat, there are a bunch of undocumented RPC methods that
we don't support yet.

Here is the workflow:

1. Answer all the application considerations above.
2. Answer all the technical considerations above.
3. Work out what methods you need to call, what data they return, and
   if it exists for all the versions you want to support (if you
   definitely must have it).

The repository is here: https://github.com/rust-bitcoin/corepc

We want your client to be bullet proof so the `types` crate is
designed a little strangely. Bare with me.

We provide version specific types e.g., `types::v28::SomeCoreMethod`
is the data returned by Bitcoin Core version 28 for the
`somecoremethod` method. Note however that the version specific types
only use rust standard types e.g., `String`, `u64` etc. If you want
the fields strongly typed using `rust-bitcoin` types then we provide a
version non-specific counterpart in the `model` module e.g.,
`model::SomeCoreMethod`. 

WTF, you idiot, why did you do that? Just deserialize the JSON into
`rust-bitcoin` types when getting back from Core?

The reason we did this is the bullet proof thing

- I don't want bugs in `rust-bitcoin`'s `serde` impls breaking your
  client in production.
- I don't want data formatting problems in Core's API breaking your
  client in production.
- And when one or both of these things does happen, and trust me it
  will, I want your team to have at least some hope of debugging it.

Every version specific type implements a function `into_model()` that
can be called to convert it into a version non-specific type of the
same name. That type will have optional fields if things have changed
since v17. It will have `rust-bitcoin` types for any field that
requires it. And it will give you a decent debugging message when it
fails. And you'll know if the failure was with the Core data or with
the conversion logic.

So there you have it. If you don't like it that is ok. If you do like
it use it, it's free, it's open, and it is done.

Thanks for reading.
