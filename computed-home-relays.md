NIP Computed Home Relays
======

## Abstract

Finding a user's [NIP-65](nostr:naddr1qqrxu6ts95mr2q3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxzh6n870) relay list requires knowing where to look. This NIP derives that place from the pubkey itself: every client using the same relay list computes the same few public relays, the user's home relays, where their `kind:10002` is published.

## Home Relays

Anyone can publish a relay list: a generic list named `home-relays`. Its relays are the candidates.

The distance between a user and a candidate is `sha256(npub) XOR sha256(url)`, compared as 256-bit integers, with `npub` the user's bech32 pubkey and `url` the normalized relay URL without trailing `/`, both hashed as UTF-8 text. The home relays are the 8 closest candidates.

## Example

For `npub1jlp4rzsraw0nvvuyavgf2z497juw7y44v40ch23lnwzlnre3wuusjhmu65` and this list:

```json
{
  "kind": 36205,
  "tags": [
    ["d", "home-relays"],
    ["relay", "wss://indexer.coracle.social"],
    ["relay", "wss://nostr-pub.wellorder.net"],
    ["relay", "wss://nostr.bitcoiner.social"],
    ["relay", "wss://nostr.data.haus"],
    ["relay", "wss://nostr.oxtr.dev"],
    ["relay", "wss://offchain.pub"],
    ["relay", "wss://purplepag.es"],
    ["relay", "wss://relay.angor.io"],
    ["relay", "wss://relay.coinos.io"],
    ["relay", "wss://relay.ditto.pub"],
    ["relay", "wss://relay.nos.social"],
    ["relay", "wss://relay.nostr.net"],
    ["relay", "wss://relay.nostrplebs.com"],
    ["relay", "wss://relay.primal.net"],
    ["relay", "wss://user.kindpag.es"],
    ["relay", "wss://yabu.me"]
  ],
  "content": ""
}
```

| Relay | Distance |
|-------|----------|
| `wss://relay.angor.io` | `0d3d6817a20a2a741ccdad2a51b45e5e3d5e143321dce0137eda8f1673accf5b` |
| `wss://relay.primal.net` | `10b0aebc4d53b5a8863869c919ffb8246c63071964659c0e3c3707ba950a0f25` |
| `wss://yabu.me` | `160bf94f3caa56426f00d7d9e0225f0b9468efc4a0299715649bd269c8384a72` |
| `wss://relay.ditto.pub` | `1bfbd0d898ca6a4f911f025444962190e80e73701babf88d6f227e36ace2b503` |
| `wss://purplepag.es` | `2845af6b6cbb981519e59d036dfed37e0247c6dcec0075ab78438212ad80b840` |
| `wss://relay.coinos.io` | `2901f9706f074b4f1cc84457f2f3380a8d0c40fda3eb4f1fd020c097fdfe184e` |
| `wss://nostr.data.haus` | `2bfd07dbf34efe6fa1c4e5fab03c3d841341effebfc3f02d9e1f9bdddd33c928` |
| `wss://indexer.coracle.social` | `4b4a05d78b9cf545c01218671f3740dd977da1c0b863ac43582199795cad3022` |

## Client Behavior

1. A client SHOULD keep the user's latest `kind:10002` on their home relays from every relay list it uses.
2. To find the `kind:10002` of a pubkey with no other hint, a client queries that pubkey's home relays from one of its relay lists.
