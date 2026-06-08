# Mail Contacts

`draft` `optional`

This document defines a portable Nostr list for mail contacts.

The list is stored with the append-only lists model defined by the
Append-Only Lists NIP, using kinds `1990` and `1991`.

## List name

Mail contacts are stored in the list named:

```text
mail/contacts
```

Clients SHOULD use this exact `d` tag value so different mail clients can share
the same contacts list.

## Entries

Each contact is represented by one append-only list entry:

```text
tag:   m
value: <encoded mailbox>
```

The `value` is a single encoded mailbox string containing an email address and
an optional display name. It SHOULD be compatible with common RFC 5322 mailbox
parsers.

Examples:

```text
alice@example.com
"Alice Example" <alice@example.com>
npub1example...@nostr
"Bob" <npub1example...@nostr>
```

## Privacy

Contacts SHOULD be stored as private append-only list entries, using the
NIP-44 encrypted content inherited from NIP-51.

Clients MUST be able to read public `m` entries too, but SHOULD NOT publish
mail contacts publicly unless the user explicitly asks for that.

## Adding A Contact

To add a contact, publish a kind `1990` event with:

```json
{
  "kind": 1990,
  "tags": [
    ["d", "mail/contacts"]
  ],
  "content": "[[\"m\",\"\\\"Alice Example\\\" <alice@example.com>\"]]"
}
```

The example above shows the private form, where the `m` tag tuple is carried
inside encrypted content. Public entries use the same tuple as a normal event
tag:

```json
["m", "\"Alice Example\" <alice@example.com>"]
```

## Removing A Contact

To remove a contact, publish a kind `1991` event for the exact same `m`
entry value:

```json
{
  "kind": 1991,
  "tags": [
    ["d", "mail/contacts"]
  ],
  "content": "[[\"m\",\"\\\"Alice Example\\\" <alice@example.com>\"]]"
}
```
