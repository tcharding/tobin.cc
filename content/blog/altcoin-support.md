+++
title = "rust-bitcoin - Altcoin Support"
date = 2021-11-10T13:21:39+11:00
draft = false
+++

Writing bitcoin wallets in Rust is easy with
[bdk](https://github.com/bitcoindevkit/bdk/), what if we want to write a wallet
for one of the altcoins that forked off from `bitcoind`. Well with some research
and a little bit of hacking we are off to the races.

<!--more-->

Recently I was tasked with adding support to a software wallet for two coins
that both forked the `bitcoind` codebase, Dogecoin and Litecoin. We already have
support for Bitcoin by way of `bdk` and since `bdk` is super nice to use I
wanted to use it for the others also.

`bdk`, like most software in Rust that works with Bitcoin, uses a bunch of
repositories that all live under the
[rust-bitcoin](https://github.com/rust-bitcoin) organisation on GitHub (links
here are to my forks). 

- [bitcoin_hashes](https://github.com/tcharding/bitcoin_hashes)
- [rust-secp256k1](https://github.com/tcharding/rust-secp256k1)
- [rust-bitcoin](https://github.com/tcharding/rust-bitcoin)
- [rust-miniscript](https://github.com/tcharding/rust-miniscript)
- [rust-bitcoincore-rpc](https://github.com/tcharding/rust-bitcoincore-rpc)
  (Used by test code in `bdk`) - I won't discuss it further.

Turns out that to get altcoin support working I only had to hack `rust-bitcoin`,
`rust-miniscript`, and `bdk`.

## Address prefixes

I'm no expert on Litecoin or Dogecoin, as far as I could tell the only thing
that has changed, from the perspective of a wallet developer, was the address
prefixes used by the various chains. It took me a bit of digging around to find
these but eventually I found `src/chainparams.cpp` and came up with this table:

```
| Type           | BTC mainnet | BTC testnet | DOGE mainnet | DOGE testnet | LTC mainnet | LTC testnet |
|----------------+-------------+-------------+--------------+--------------+-------------+-------------|
| PUBKEY_ADDRESS | 0 - 0x00    | 111 - 0x6f  | 30 - 0x1e    | 113 - 0x71   | 48 - 0x30   | 111 - 0x6f  |
| SCRIPT_ADDRESS | 5 - 0x05    | 196 - 0xc4  | 22 - 0x16    | 196 - 0xc4   | 0x32 (0x05) | 58 - 0x3a   |
| SECRET_KEY     | 128 - 0x80  | 239 - 0xef  | 158 - 0x9e   | 241 - 0xf1   |             |             |
| EXT_PUBLIC_KEY | 0488B21E    | 043587CF    | 02FACAFD     | 043587CF     |             |             |
| EXT_SECRET_KEY | 0488ADE4    | 04358394    | 02FAC398     | 04358394     |             |             |

```

Its not complete, but that was enough to get the functionality I needed. Now
herein lies an insidious problem. The astute among you might have realised that
there are duplicate values in the table above, more on this below.

In order to work with addresses in `rust-bitcoin` we need to be able to go to
and from strings.

The `Address` struct currently looks like this:

```
/// A Bitcoin address
pub struct Address {
    /// The type of the address
    pub payload: Payload,
    /// The network on which this address is usable
    pub network: Network,
}
```

At first I just wanted to throw a `blockchain` field in there and then use that
to map between prefixes, but since I'm not as clever as I wish I was I didn't
notice the duplicate value problem until after I'd written that code. So, once
I'd run into that wall, I added a `prefix` field instead.

```
    ...
    /// Any prefix data we need to be able to serialize the address.
    pub prefix: Prefix,
}

/// Prefix data required to serialize an address.
// This is tightly coupled with the Network, if someone mutates an address by
// changing the network without updating the prefix bad things will happen.
pub enum Prefix {
    /// Pubkey hash prefix byte.
    Pubkey(u8),
    /// Script hash prefix byte.
    Script(u8),
    /// Segwit prefix characters e.g., "bc"
    Segwit(String),
}
```

From the code comment on `Prefix` we can already see that this is not going to
be pretty. 

For the record, this work has zero chance of upstreaming, and I would not want
to push it up anyways. I'm thankful that I was able to use these libraries to
get the job done but I in no way condone adding altcoin support to the Bitcoin
stack.

Now we can add the `Blockchain` struct
```
/// Supported blockchains.
pub enum Blockchain {
    /// The Bitcoin blockchain.
    Bitcoin,
    /// The Dogecoin blockchain.
    Dogecoin,
    /// The Litecoin blockchain.
    Litecoin,
}
```

As we can see above an `Address` has two parts, the `Payload` and the `Network`.
We can work out the prefix as long as we have both of these and the chain we are
on. To stringify an `Address` we need the prefix.

Now to go the other way, from an address string to an `Address` we just parse
the prefix.

Walah, that's more or less all there is to it. Now its just a matter of updating
the whole stack to use the new `Address` type and constructors. None of the
other repositories required any interesting or difficult work, you can check
them out by fetching the `altcoin-support` branch of each repo on my GitHub
account (linked above).

I hope this is useful if you find yourself supporting Bitcoin clones in wallet
software written in Rust. Feel free to open PRs adding support for additional
coins.

Thanks!
