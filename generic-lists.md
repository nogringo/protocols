NIP Generic Lists
======

## Abstract

NIP-51 assigns a new event kind to every list. A kind requires a merged NIP to be assigned, and any draft using an unassigned one can be overwritten by a later spec.

This NIP defines a single addressable kind where the list is named by its `d` tag, so a new list type costs no kind allocation and cannot be taken by someone else.

## Event Definition

Generic lists use `kind: 36205`. The `d` tag names the list. Tag layout and encrypted `.content` follow NIP-51.

```json
{
  "kind": 36205,
  "tags": [
    ["d", "<list-name>"],
    ["p", "<pubkey>"],
    ["e", "<event-id>"]
  ],
  "content": "<nip-44 encrypted tags>"
}
```

## Known lists

| name | description | spec |
| ---- | ----------- | ---- |
| `trusted-auth-relays` | relays to authenticate to without asking the user | Trusted Auth Relays |
| `trusted-domains` | domains to open without a security warning | Trusted Domains |
