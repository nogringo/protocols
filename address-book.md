# Address Book

`draft` `optional`

This document defines a portable, private address book.

The goal is to let mail clients, contact apps, phone/call apps, calendar apps,
and other personal information tools share the same contact cards without being
tied to a single application.

## Motivation

Nostr already has NIP-02 follow lists, often called "contact lists", but those
lists describe Nostr profiles a user follows. They are not a general address
book.

A general contact may contain names, email addresses, phone numbers, postal
addresses, organizations, notes, avatars, Nostr public keys, and other contact
methods. This protocol defines a Nostr-native storage layer for that broader
address-book use case.

## Event Kind

Contact cards are stored as addressable events.

```text
kind: 38522
```

## Addressing

Each contact card event MUST have a `d` tag containing the vCard `UID` of the
contact:

```json
["d", "urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6"]
```

## Contact Card Format

Contact cards MUST be encoded as vCard 4.0 objects as defined by RFC 6350.

Clients MAY import older vCard versions, jCard, or other contact formats, but
the canonical payload stored in contact card events MUST be vCard 4.0.

Each card MUST contain a stable `UID`. The `UID` MUST match the event `d` tag.

Clients creating a new contact SHOULD generate a UUID-based `UID`, for example:

```text
urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6
```

Example vCard value:

```text
BEGIN:VCARD
VERSION:4.0
UID:urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6
FN:Alice Example
N:Example;Alice;;;
EMAIL;TYPE=work;PREF=1:alice@example.com
TEL;TYPE=cell,voice:tel:+33123456789
IMPP:nostr:npub1example...
REV:20260607T120000Z
END:VCARD
```

## Privacy

Contact cards SHOULD be private.

The event `content` SHOULD contain the vCard 4.0 payload encrypted to the
author using NIP-44 self-encryption.

Clients MUST NOT publish contact cards in plaintext unless the user explicitly
asks for that.

Clients SHOULD generate opaque UUID-based `UID` values so the public `d` tag
does not leak personal information.

## Event Structure

Example contact card event:

```json
{
  "kind": 38522,
  "tags": [
    ["d", "urn:uuid:f81d4fae-7dec-11d0-a765-00a0c91e6bf6"]
  ],
  "content": "<nip44-encrypted vCard 4.0>"
}
```

The decrypted `content` MUST be a single vCard 4.0 object.

## Adding a Contact

To add a contact, publish a contact card event with a new `UID`.

Applications MAY create a minimal contact when the user saves a domain-specific
recipient. For example, a mail client can create a vCard containing only `UID`,
`FN`, `N`, and one `EMAIL` property.

## Updating a Contact

To update a contact, publish a newer contact card event with the same `UID` in
the `d` tag and in the decrypted vCard.

Applications SHOULD preserve unknown vCard properties and parameters when
editing a contact.

## Deleting a Contact

To delete a contact, clients SHOULD publish a NIP-09 deletion event for the
latest known contact card event id.

Publishing a newer non-deleted card with the same `UID` restores the contact.

## Private Media

Contact cards MUST remain small enough for ordinary relay limits.

Clients SHOULD NOT embed large binary media, such as contact photos, directly
inside vCards.

Public media MAY be referenced with vCard `PHOTO` URI values.

Private media SHOULD be stored as encrypted blobs outside the contact card. The
contact card SHOULD contain only the URI and the metadata required to retrieve
and decrypt the blob. Nostr clients SHOULD prefer user-controlled media storage
such as Blossom when possible.

If a client does not implement encrypted private media, it SHOULD keep private
contact photos local instead of publishing them in plaintext.
