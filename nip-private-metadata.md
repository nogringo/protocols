NIP Private Metadata
======

This NIP defines `kind:16789`, a replaceable event carrying profile metadata that only its author can read, mirroring `kind:0`. It lets a user describe their own key for themselves without publishing that information.

## Event

```jsonc
{
  "kind": 16789,
  "pubkey": "<author pubkey>",
  "created_at": 1757800000,
  "tags": [],
  "content": "<NIP-44 ciphertext>",
  "id": "...",
  "sig": "..."
}
```

`content` is a stringified JSON object encrypted with [NIP-44](nostr:naddr1qvzqqqrcvypzq2eeknl7v2fnm7tsuxfkds3vrcyjl9fls070a465urcy6k3mgk0eqqrxu6ts956rgat9m6z), using the conversation key computed from the author's own private and public key.

## Content

The decrypted object uses the fields defined for `kind:0` in [NIP-01](nostr:naddr1qqrxu6ts95crzq3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxz62xsjt) and [NIP-24](nostr:naddr1qqrxu6ts95ergq3q9vumfllx9yeal9cwrymxcgkpuzf0j5lc8l876a2wpuzdtga5t8usxpqqqpuxz6lkr7v).

```json
{
  "name": "Amazon",
  "about": "Key used for shopping accounts",
  "picture": "https://example.com/amazon.png"
}
```

When rewriting the event, clients MUST keep fields they do not understand.

## Client behavior

Private metadata is displayed only to its author, that is, in a client that holds a signer for the event's pubkey.

For each field, clients SHOULD display the private value when present and fall back to the `kind:0` value otherwise.

Clients MUST NOT copy private values into anything visible to others, such as `kind:0` or any published event.
