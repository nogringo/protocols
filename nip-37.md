NIP-37
======

Draft Wraps
-----------

`draft` `optional`

This NIP defines kind `31234` as an encrypted storage for unsigned draft events of any other kind. 

The draft is JSON-stringified, [NIP44-encrypted](nostr:naddr1qvzqqqrcvypzq2eeknl7v2fnm7tsuxfkds3vrcyjl9fls070a465urcy6k3mgk0eqqrxu6ts956rgat9m6z) to the signer's public key and placed inside the `.content`.

`k` tags identify the kind of the draft. 

```js
{
  "kind": 31234,
  "tags": [
    ["d", "<identifier>"],
    ["k", "<kind of the draft event>"], // required
    ["expiration", "now + 90 days"] // recommended
  ],
  "content": nip44Encrypt(JSON.stringify(draft_event)),
  // other fields
}
```

A blanked `.content` field signals that the draft has been deleted. 

[NIP-40](nostr:naddr1qqrxu6ts956rqq3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxzh0r5jg) `expiration` tags are recommended. 

Clients SHOULD publish kind `31234` events to relays listed on kind `10013` below.

## Checkpoints
`kind:1234` defines checkpoints that belong to a parent `kind:31234` event. These can serve to provide a revision history of a parent event.

```js
{
  "kind": 1234,
  "tags": [
    ["a", "31234:<pubkey>:<identifier>"]
  ],
  "content": nip44Encrypt(JSON.stringify(draft_event)),
}
```

## Relay List for Private Content

Kind `10013` indicates the user's preferred relays to store private events like Draft Wraps. 

The event MUST include a list of `relay` URLs in private tags. Private tags are JSON Stringified, [NIP44-encrypted](nostr:naddr1qvzqqqrcvypzq2eeknl7v2fnm7tsuxfkds3vrcyjl9fls070a465urcy6k3mgk0eqqrxu6ts956rgat9m6z) to the signer's keys and placed inside the .content of the event.

```js
{
  "kind": 10013,
  "tags": [],
  "content": nip44Encrypt(
    JSON.stringify(
      [
        ["relay", "wss://myrelay.mydomain.com"]
      ]
    )
  )
  //...other fields
}
```

It's recommended that Private Storage relays SHOULD be [NIP-42](nostr:naddr1qvzqqqrcvypzq2eeknl7v2fnm7tsuxfkds3vrcyjl9fls070a465urcy6k3mgk0eqqrxu6ts956ryv4r3t9)-authed and only serve events to the authed user who uploaded them.

Clients MUST publish kind `10013` events to the author's [NIP-65](nostr:naddr1qqrxu6ts95mr2q3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxzh6n870) `write` relays. 
