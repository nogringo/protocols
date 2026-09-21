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
| `signature` | string | Email signature appended to outgoing emails |
| `bridges` | string[] | List of preferred bridge domains |
| `identities` | string[] | List of user-defined "From" identities in RFC 2822 format |
| `folders` | object[] | User folders, see [Folders and Tags](#folders-and-tags) |
| `tags` | object[] | User tags, see [Folders and Tags](#folders-and-tags) |

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
| `position` | integer | no | Sort key, ascending. When absent, the entry sorts last. Ties are broken by `name` |
| `match` | object | no | Condition, see [Matched Entries](#matched-entries) |

Each array is ordered on its own.

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
  "position": 0,
  "match": { "from": ["github.com"] }
}
```

An email is in exactly one folder, resolved in this order:

1. Its `folder:` label, when it carries one.
2. The first entry of `folders` whose `match` it matches, in `position` order.
3. Its natural mailbox.

A tag holds every email carrying its `tag:<id>` label, plus every email its `match` matches.

A folder or a tag lists the emails it holds, except those in `folder:trash` and `folder:spam`.

### Example (Decrypted)

```json
{
  "signature": "Sent via Nostr Mail",
  "bridges": ["nostr.mail", "bridge.example.com"],
  "identities": [
    "Alice Real <npub1abc...@nostr.mail>",
    "<npub1abc...@bridge.com>",
    "Pseudo <alice@example.com>"
  ],
  "folders": [
    { "id": "9f2c1a7b4d3e5f60", "name": "Invoices", "color": "#C86432", "position": 0 },
    {
      "id": "e4d1907b23c6af58",
      "name": "GitHub",
      "position": 1,
      "match": { "from": ["github.com"] }
    }
  ],
  "tags": [
    { "id": "4b81d0e7a5c39f12", "name": "Urgent", "position": 0 }
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
