# Snapshots of Replaceable Events

`draft` `optional`

Kind `1349` preserves one version of a replaceable or addressable event by wrapping it in a
regular event, which relays keep.

## Motivation

Relays keep only the latest replaceable or addressable event, so an author can rewrite a
document and leave no trace of what it said before. Approvals and highlights point at a
coordinate rather than at an id, so they silently transfer to whatever replaces the text
they were given to.

## The event

`content` is the wrapped event, serialized as JSON exactly as it appeared on the wire.
Anyone may publish a snapshot: the inner signature proves who wrote the document, the outer
one proves only who archived it.

| Tag | Description                                                                              |
| --- | ---------------------------------------------------------------------------------------- |
| `e` | Id of the wrapped event.                                                                  |
| `k` | Kind of the wrapped event.                                                                |
| `p` | Pubkey of the wrapped event's author.                                                     |
| `a` | Coordinate of the wrapped event, with an empty `d` for kinds 10000 to 19999.               |

```jsonc
{
  "kind": 1349,
  "content": "{\"id\":\"a1b2...\",\"pubkey\":\"9f8e...\",\"created_at\":1771547830,\"kind\":30817,\"tags\":[[\"d\",\"my-spec\"]],\"content\":\"# My Spec\",\"sig\":\"c3d4...\"}",
  "tags": [
    ["e", "a1b2..."],
    ["k", "30817"],
    ["p", "9f8e..."],
    ["a", "30817:9f8e...:my-spec"]
  ]
}
```

## Validation

The tags are written by the publisher, who may be anyone. Without checking them, anyone
could attach arbitrary events to another document's history.

Clients MUST discard a snapshot unless:

1. `content` parses as an event and its signature verifies;
2. `e`, `k`, `p` and `a` agree with the wrapped event;
3. the wrapped kind is replaceable or addressable.

Order versions by the wrapped event's `created_at`, not the snapshot's, which is only the
time of archival. Deduplicate on the wrapped event's id; several archivers wrapping the
same version is expected.

## Querying

```json
{ "kinds": [1349], "#a": ["30817:<pubkey>:<d>"] }
```

A snapshot proves a version existed. It never proves a set of them is complete, so a
history is what was observed, not what happened.
