NIP Computed Home Relays
======

## Abstract

Finding a user's [NIP-65](nostr:naddr1qqrxu6ts95mr2q3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxzh6n870) relay list requires knowing where to look. This NIP derives that place from the pubkey itself: every client computes the same few public relays, the user's home relays, where their `kind:10002` is published.

## Home Relays

Candidates are the `wss://` relays with a [NIP-66](nostr:naddr1qqrxu6ts95mrvq3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxzujryhy) `kind:30166` from the last 24 hours, except those tagged `["R", "auth"]`, `["R", "payment"]` or `["R", "writes"]`. IP addresses are excluded, and each registrable domain (ICANN section of the [Public Suffix List](https://publicsuffix.org/)) keeps a single candidate: its shortest URL without trailing `/`, ties broken alphabetically.

The distance between a user and a candidate is `sha256(npub) XOR sha256(domain)`, compared as 256-bit integers, with `npub` the user's bech32 pubkey and `domain` the candidate's registrable domain in lowercase ASCII, both hashed as UTF-8 text. The home relays are the 8 closest candidates.

## Example

For `npub1jlp4rzsraw0nvvuyavgf2z497juw7y44v40ch23lnwzlnre3wuusjhmu65`:

| Relay | Domain | Distance |
|-------|--------|----------|
| `wss://promenade.fiatjaf.com` | `fiatjaf.com` | `681dbcdfbd3f0bc404c56e5b39bcbf17f43abf34ab371191191b1ed0f1ecb82d` |
| `wss://relay.primal.net` | `primal.net` | `7413d508501106245f16daa065d84d6f4ada52b2bac0555512569479b7b03d11` |
| `wss://relay.ditto.pub` | `ditto.pub` | `e5423a90c958dd6cdf4a089e5a4e0c01dab20157a80b03ff998051de1ec791ad` |

## Client Behavior

1. A client SHOULD keep the user's latest `kind:10002` on their home relays.
2. To find the `kind:10002` of a pubkey with no other hint, a client queries that pubkey's home relays.
