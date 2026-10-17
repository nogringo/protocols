# Nostr Mail Settings

This document defines the protocol for storing and synchronizing user settings across devices in Nostr Mail using NIP-78 application-specific data.

## Overview

User settings are stored as replaceable parameterized events (kind 30078). This enables seamless synchronization across multiple devices while maintaining privacy through encryption.

## Event Kind

### Kind 30078: Application-Specific Data

Settings use NIP-78 replaceable parameterized events with the `d` tag for namespacing.

## Public Settings

Public settings are stored unencrypted and can be read by anyone.

```json
{
  "kind": 30078,
  "pubkey": "<user_pubkey>",
  "tags": [["d", "nostr-mail/settings"]],
  "content": "{\"dm_copy\": true, \"prefer_nostr\": true}"
}
```

### Fields

| Field | Type | Description |
|-------|------|-------------|
| `dm_copy` | boolean | Request bridges to send a DM copy of incoming emails |
| `prefer_nostr` | boolean | Deliver email for this key's NIP-05 addresses over Nostr rather than SMTP. |

When updating either settings event, a client MUST keep the fields of `content` it does not change.

## Transport Preference

For a recipient given as an email address `name@domain`:

1. Resolve it through NIP-05. If it does not resolve to a pubkey, deliver over SMTP.
2. Fetch the pubkey's public settings from its NIP-65 write relays.
3. If `prefer_nostr` is `true`, deliver over Nostr. Otherwise, deliver over SMTP.

A failed lookup counts as `false`.

## Private Settings

Private settings MUST be encrypted to self using NIP-44.

```json
{
  "kind": 30078,
  "pubkey": "<user_pubkey>",
  "tags": [["d", "nostr-mail/settings/private"]],
  "content": "<nip44_encrypted_json>"
}
```

### Decrypted Content

After decryption, the content contains:

| Field | Type | Description |
|-------|------|-------------|
| `signature` | string | Plain text email signature, see [Legacy Signature](#legacy-signature) |
| `signatures` | object[] | Email signatures, see [Signatures](#signatures) |
| `default_signature` | object | Signatures used when the identity has no choice, see [Signature Choice](#signature-choice) |
| `identity_signatures` | object | Signatures chosen per identity, see [Signature Choice](#signature-choice) |
| `signature_images` | object | Images of the signatures, see [Signature Images](#signature-images) |
| `bridges` | string[] | List of preferred bridge domains |
| `identities` | string[] | List of user-defined "From" identities in RFC 5322 format |
| `folders` | object[] | User folders, see [Folders and Tags](#folders-and-tags) |
| `tags` | object[] | User tags, see [Folders and Tags](#folders-and-tags) |

### Signatures

Each entry:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | 16 lowercase hex characters, unique within `signatures` |
| `name` | string | yes | 1 to 64 characters after trimming, unique case-insensitively within `signatures` |
| `html` | string | yes | HTML fragment appended to the email |

Any signature can be used with any identity.

When a client rewrites an entry, it MUST keep the fields of that entry it does not change.

An `<img>` whose `src` equals a key of `signature_images` shows that image.

```json
{
  "id": "a1b2c3d4e5f60718",
  "name": "Full",
  "html": "<p>Alice Real</p><img src=\"3f1a9c0e7b2d4f6a8c5e1b9d7f3a2c4e6b8d0f1a3c5e7b9d2f4a6c8e0b1d3f5a\" alt=\"Example Corp\">"
}
```

### Signature Choice

A choice is an object:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `new` | string | no | `id` of the signature for a new email |
| `reply` | string | no | `id` of the signature for a reply or a forward |

An empty string means no signature. An `id` with no entry in `signatures` means no signature.

`identity_signatures` maps an entry of `identities`, as the exact string, to a choice. When the identity is absent from the map, or a field is absent from its choice, the client uses that field of `default_signature`. When the field is absent from `default_signature` too, the email gets no signature.

When a client rewrites an entry of `identities`, it MUST move its key in `identity_signatures`.

```json
{
  "default_signature": { "new": "a1b2c3d4e5f60718", "reply": "9e8d7c6b5a493827" },
  "identity_signatures": {
    "Pseudo <alice@example.com>": { "new": "9e8d7c6b5a493827" },
    "<npub1abc...@bridge.com>": { "new": "", "reply": "" }
  }
}
```

### Legacy Signature

When `signatures` is absent, clients append `signature`, as plain text, to every outgoing email.

### Signature Images

Each image is encrypted with AES-GCM, using a random 256-bit key and a random 96-bit nonce. The encrypted blob is uploaded to the Blossom servers of the user's kind 10063 list.

`signature_images` maps the SHA-256 hex of each encrypted blob to an entry:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `encryption_algorithm` | string | yes | `aes-gcm` |
| `decryption_key` | string | yes | Base64 key |
| `decryption_nonce` | string | yes | Base64 nonce |
| `mime_type` | string | yes | Media type of the decrypted image, such as `image/png` |
| `size` | integer | yes | Size of the encrypted blob in bytes |
| `dim` | string | no | Pixel size of the decrypted image, as `<width>x<height>` |

A client reads an image by downloading its key from the user's Blossom servers, checking its SHA-256, then decrypting it.

When sending, a client adds each image the signature shows as an inline MIME part with its own `Content-ID`, and points the HTML body at it with a `cid:` URL.

When a key leaves `signature_images`, the client SHOULD delete its blob from the user's Blossom servers.

### Identities

Identities are user-defined "From" addresses stored as RFC 5322 formatted strings. Each entry can be used directly in the `From:` header without any transformation.

**Format examples:**
- `"Alice Real <npub1abc...@nostr.mail>"`: name + address
- `"npub1abc...@bridge.com"`: address only (no name)
- `"Pseudo <alice@example.com>"`: custom name + legacy email

**Behavior:**
- If `identities` is empty or absent, clients SHOULD auto-generate available addresses from `npub@nostr` and configured bridges
- The **first identity** (index 0) is the default "From" address

### Folders and Tags

`folders` and `tags` name the user folders and user tags an email carries as `folder:<id>` and `tag:<id>` labels. See [Nostr Mail Labels](nostr-mail-labels.md).

Each entry:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | yes | 16 lowercase hex characters, unique across `folders` and `tags` |
| `name` | string | yes | 1 to 64 characters after trimming, unique case-insensitively within its own array |
| `color` | string | no | `#RRGGBB`. When absent, clients derive one from `id` |
| `match` | object | no | Condition, see [Matched Entries](#matched-entries) |

Each array is ordered on its own, in array order.

Entries written by earlier clients may carry `position`, an integer sort key. When an entry of an array has one, clients order that array by `position`, ascending, with entries without one last and ties broken by `name`. A client that writes the order of an array removes `position` from all its entries.

Folders are flat. A folder has no parent.

When a client rewrites an entry, it MUST keep the fields of that entry it does not change.

An email carrying a `folder:<id>` or `tag:<id>` with no entry keeps it. Clients SHOULD display it under its id.

### Matched Entries

A `match` holds an email without any label event being written.

| Field | Type | Matches when |
|-------|------|--------------|
| `from` | string[] | The sender address equals one entry, or ends with `@` or `.` followed by one entry |
| `subject` | string[] | The subject contains one entry |
| `has_attachment` | boolean | The email carries at least one attachment, or none when false |

Fields are combined with AND, the entries of one field with OR. Text is compared case-insensitively. A `match` with no field matches no email.

```json
{
  "id": "e4d1907b23c6af58",
  "name": "GitHub",
  "match": { "from": ["github.com"] }
}
```

An email is in exactly one folder, resolved in this order:

1. Its `folder:` label, when it carries one.
2. The first entry of `folders` whose `match` it matches.
3. Its natural mailbox.

A tag holds every email carrying its `tag:<id>` label, plus every email its `match` matches.

A folder or a tag lists the emails it holds, except those in `folder:trash` and `folder:spam`.

### Example (Decrypted)

```json
{
  "signatures": [
    {
      "id": "a1b2c3d4e5f60718",
      "name": "Full",
      "html": "<p>Alice Real</p><img src=\"3f1a9c0e7b2d4f6a8c5e1b9d7f3a2c4e6b8d0f1a3c5e7b9d2f4a6c8e0b1d3f5a\" alt=\"Example Corp\">"
    },
    { "id": "9e8d7c6b5a493827", "name": "Short", "html": "<p>Alice</p>" }
  ],
  "default_signature": { "new": "a1b2c3d4e5f60718", "reply": "9e8d7c6b5a493827" },
  "identity_signatures": {
    "Pseudo <alice@example.com>": { "new": "9e8d7c6b5a493827" }
  },
  "signature_images": {
    "3f1a9c0e7b2d4f6a8c5e1b9d7f3a2c4e6b8d0f1a3c5e7b9d2f4a6c8e0b1d3f5a": {
      "encryption_algorithm": "aes-gcm",
      "decryption_key": "q7N2v0kXhF4mY8cTz1bR6wLpJ3sD9eGu5aKiO0nVxMc=",
      "decryption_nonce": "bF3kT8wQz1Lm4Y9c",
      "mime_type": "image/png",
      "size": 8231,
      "dim": "600x120"
    }
  },
  "bridges": ["nostr.mail", "bridge.example.com"],
  "identities": [
    "Alice Real <npub1abc...@nostr.mail>",
    "<npub1abc...@bridge.com>",
    "Pseudo <alice@example.com>"
  ],
  "folders": [
    { "id": "9f2c1a7b4d3e5f60", "name": "Invoices", "color": "#C86432" },
    {
      "id": "e4d1907b23c6af58",
      "name": "GitHub",
      "match": { "from": ["github.com"] }
    }
  ],
  "tags": [
    { "id": "4b81d0e7a5c39f12", "name": "Urgent" }
  ]
}
```

### Example (Encrypted Event)

```json
{
  "kind": 30078,
  "tags": [["d", "nostr-mail/settings/private"]],
  "content": "<nip44_encrypted_blob>"
}
```
