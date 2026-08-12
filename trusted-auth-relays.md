NIP Trusted Auth Relays
======

## Abstract

Answering a [NIP-42](https://github.com/nostr-protocol/nips/blob/master/42.md) `AUTH` challenge reveals the user's pubkey to the relay, but asking the user on every challenge is unusable, so most clients authenticate automatically.

This NIP defines a list of the relays the user trusts with their identity. Anything that handles `AUTH` challenges (client, extension, remote signer) can read this list and authenticate without prompting.

## Event Definition

This list is a generic list named `trusted-auth-relays`.

`["relay", "<relay-url>"]`: a relay the user is willing to authenticate to without being asked. Matched against the relay URL after normalization; subdomains are NOT automatically trusted.

```json
{
  "kind": 36205,
  "tags": [
    ["d", "trusted-auth-relays"],
    ["relay", "wss://relay.example.com/"]
  ],
  "content": "<nip-44 encrypted tags>"
}
```

## Client Behavior

On an `AUTH` challenge from a listed relay, the client MAY sign the `kind: 22242` event without prompting the user. Otherwise it MUST NOT authenticate silently: it SHOULD ask the user, or refuse.
