NIP Computed Home Relays
======

## Abstract

Finding a user's [NIP-65](https://github.com/nostr-protocol/nips/blob/master/65.md) relay list requires knowing where to look. This NIP derives that place from the pubkey itself: every client computes the same few public relays, the user's home relays, where their `kind:10002` is published.

## Home Relays

Candidates are the `wss://` relays with a [NIP-66](https://github.com/nostr-protocol/nips/blob/master/66.md) `kind:30166` from the last 24 hours, except those tagged `["R", "auth"]`, `["R", "payment"]` or `["R", "writes"]`. IP addresses are excluded, and each registrable domain (ICANN section of the [Public Suffix List](https://publicsuffix.org/)) keeps a single candidate: its shortest URL, ties broken alphabetically.

The distance between a user and a relay is `sha256(npub) XOR sha256(url)`, compared as 256-bit integers, with `npub` the user's bech32 pubkey and `url` the relay URL without trailing `/`, both hashed as UTF-8 text. The home relays are the 8 closest candidates.

## Example

For `npub1jlp4rzsraw0nvvuyavgf2z497juw7y44v40ch23lnwzlnre3wuusjhmu65`:

| Relay | Distance |
|-------|----------|
| `wss://relay.primal.net` | `10b0aebc4d53b5a8863869c919ffb8246c63071964659c0e3c3707ba950a0f25` |
| `wss://relay.ditto.pub` | `1bfbd0d898ca6a4f911f025444962190e80e73701babf88d6f227e36ace2b503` |
| `wss://promenade.fiatjaf.com` | `f404258bfe2ee7753e85256c903967052fe30bb9379e6ac90148b97fb9932db1` |

## Client Behavior

1. A client SHOULD keep the user's latest `kind:10002` on their home relays.
2. To find the `kind:10002` of a pubkey with no other hint, a client queries that pubkey's home relays.
