NIP Append-Only Lists
======

This NIP defines append-only event kinds for list membership, as a complement to NIP-51. Tag layout, encrypted-content format, and `d` tag semantics are inherited from NIP-51 unchanged; only the storage model differs.

## Motivation

NIP-51 stores lists in *replaceable* events. The implied last-write-wins semantics is poorly adapted to:

- **Local-first / offline-first workflows** — concurrent edits performed offline on different devices cannot be merged; the second publication silently overwrites the first.
- **Automated, high-volume marking** — when a client emits many list operations per session, republishing the entire list per operation is wasteful and conflict-prone.

This NIP defines append-only counterparts: each addition or removal of an entry is its own event. State is computed client-side as additions minus subsequent removals, forming an OR-Set CRDT. Operations commute, so multi-device and offline edits converge without coordination.

## Event kinds

- `kind:1990` — Add to list
- `kind:1991` — Remove from list

Both are *regular* (non-replaceable) event kinds.

## Tags and content

Tag layout and encrypted-content format follow NIP-51:

- A single `d` tag names the list. Standard NIP-51 lists have no `d` tag, so their append-only counterpart uses the kind number as the name (`["d", "10000"]` for the mute list).
- Entries are referenced via single-letter tags (`e`, `p`, `a`, `t`, …) and/or via the NIP-44 self-encrypted content, with the same plaintext format as NIP-51 (a JSON-stringified array of tag tuples).
- The effective set of entries operated on by the event is the union of the public tag references and the entries decoded from the content.

## Example

A user maintains a list of favourite fruits using hashtag references. They add three fruits:

```jsonc
{
  "kind": 1990,
  "created_at": 1715000000,
  "tags": [
    ["d", "fruits"],
    ["t", "apple"],
    ["t", "banana"],
    ["t", "cherry"]
  ],
  "content": ""
}
```

Later, they remove `banana`:

```jsonc
{
  "kind": 1991,
  "created_at": 1715100000,
  "tags": [
    ["d", "fruits"],
    ["t", "banana"]
  ],
  "content": ""
}
```

The resulting state of the `fruits` list for this author is `{apple, cherry}`.

## State computation

An entry `e` is a member of list `L` for author `P` if and only if `P` has signed at least one Add event for `(L, e)` and has not signed a Remove event for `(L, e)` with a strictly later `created_at`.

This forms an OR-Set CRDT: Add and Remove operations commute, are associative, and idempotent. Devices may emit operations offline and publish them in any order; convergence to a consistent list state is guaranteed without explicit coordination.

## Queryability

- Full state of a list: `REQ {"kinds":[1990,1991], "authors":["<pubkey>"], "#d":["<list_name>"]}`
- Membership state of a specific entry across lists: `REQ {"kinds":[1990,1991], "authors":["<pubkey>"], "#e":["<entry_id>"]}`
- Incremental sync: add `"since": <timestamp>` to any of the above.

## Relay storage

Append-only events accumulate over time. Clients SHOULD periodically consolidate (emit a fresh batch capturing the current state, then issue NIP-09 deletions on the superseded events).

## Compatibility

Complementary to NIP-51, not a replacement. The two are intended to coexist:

- NIP-51 remains appropriate for human-curated, low-volume, mono-device lists explicitly edited as a whole by the user.
- This NIP targets high-volume, automated, multi-device, and offline-first workflows.

Clients MAY mirror state between the two mechanisms for interoperability with clients that only support one.
