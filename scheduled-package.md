# Scheduled Package

`draft` `optional`

Groups one or more Scheduler DVM jobs into a single scheduled item, and stores
private context needed to display it later.

## Why

Some scheduled events are opaque to the client that scheduled them. For example,
a wrapped or encrypted event may not contain enough readable information to
show a useful scheduled item later.

Some scheduled actions also need several Scheduler DVM jobs. A package lets a
client display and cancel those jobs as one logical item.

## Manifest

A package is a NIP-37 draft event:

```json
{
  "kind": 31234,
  "pubkey": "<client_pubkey>",
  "content": "<nip37 encrypted content>",
  "tags": [
    ["d", "<package_id>"],
    ["k", "5905"],
    ["e", "<kind5905_event_id_1>"],
    ["e", "<kind5905_event_id_2>"]
  ]
}
```

| Tag | Description |
|-----|-------------|
| `d` | Stable package identifier. SHOULD be 64 random hex chars. |
| `k` | Linked event kind. MUST be `5905`. |
| `e` | Linked Scheduler DVM request ID. One tag per job. |

## Content

The draft content is application-defined. It SHOULD contain enough context for
the client to display the package later.

## Create

To create a package, the client:

1. Builds and signs every target event.
2. Builds and signs one `kind:5905` request per target event.
3. Builds and signs one `kind:31234` package manifest linking all `kind:5905`
   request IDs.
4. Publishes the requests and manifest.
