# Nostr Mail Labels

This document defines the protocol for managing email metadata (folders, read state, stars, custom tags) in Nostr Mail using NIP-32 labels.

## Overview

Email metadata is managed through NIP-32 label events (kind 1985). Each label is a separate event, allowing granular control and easy synchronization across clients.

## Namespace

All Nostr Mail labels use the namespace: `mail`

## Label Format

### Adding a Label

To add a label to an email, sign a kind 1985 event:

```json
{
  "kind": 1985,
  "pubkey": "<user_pubkey>",
  "tags": [
    ["L", "mail"],
    ["l", "<label>", "mail"],
    ["e", "<email_id>", "", "labelled"]
  ],
  "content": ""
}
```

`<email_id>` identifies the email the label applies to: the rumor id for a gift wrapped email, the event id for a public one.

Put that signed event, without a seal, in a NIP-59 gift wrap (kind 1059) addressed to the user:

```json
{
  "kind": 1059,
  "pubkey": "<random_one_time_pubkey>",
  "tags": [["p", "<user_pubkey>"]],
  "content": "<nip44(signed_1985_event_json)>"
}
```

The wrap is encrypted with NIP-44 from the one-time key to the user's own pubkey, and published to the user's DM relays (kind 10050).

### Removing a Label

To remove a label, publish a NIP-09 deletion request (kind 5) targeting the gift wrap of the label event:

```json
{
  "kind": 5,
  "pubkey": "<user_pubkey>",
  "tags": [
    ["e", "<label_gift_wrap_id>"],
    ["k", "1059"]
  ],
  "content": ""
}
```

### Reading Labels

Labels arrive through the kind 1059 subscription on `p` = the user's pubkey, alongside emails. A wrap is dispatched on the kind of the event it yields.

A kind 1985 event authored by the user and published outside a gift wrap carries the same meaning.

## Identifiers

User folders and user tags are identified by `<id>`: 16 lowercase hexadecimal characters, drawn at random when the folder or tag is created. An id never changes.

`inbox`, `sent`, `archive`, `trash` and `spam` are reserved and MUST NOT be issued as an id.

Names, colors and order live in the private settings event. See [Nostr Mail Settings](nostr-mail-settings.md).

## Standard Labels

### Folders

An email with no folder label falls to the first user folder whose condition it matches, see [Nostr Mail Settings](nostr-mail-settings.md), and to its natural mailbox when none matches: `sent` when its sender is the user, `inbox` otherwise.

Folder labels are mutually exclusive. Adding one requires removing the one already present.

| Label | Description |
|-------|-------------|
| `folder:inbox` | Email is in the inbox |
| `folder:sent` | Email is in sent |
| `folder:archive` | Email is archived |
| `folder:trash` | Email is in the trash |
| `folder:spam` | Email is marked as spam |
| `folder:<id>` | Email is in a user folder |

A `folder:trash` or `folder:archive` label event MAY name the folder the email left:

```json
["prev-folder", "<id>"]
```

Restoring the email applies that folder again.

### Read State

Emails without a read state label are considered unread (default state).

| Label | Description |
|-------|-------------|
| `state:read` | Email has been read |

### Flags

Emails without flag labels have no special flags (default state).

| Label | Description |
|-------|-------------|
| `flag:starred` | Email is starred/favorited |
| `flag:important` | Email is marked as important |

### Custom Tags

Users can create custom tags for organization. An email carries any number of them.

| Label | Description |
|-------|-------------|
| `tag:<id>` | Custom user-defined tag |

## Examples

### Move Email to Trash

```json
{
  "kind": 1985,
  "pubkey": "abc123...",
  "created_at": 1234567890,
  "tags": [
    ["L", "mail"],
    ["l", "folder:trash", "mail"],
    ["e", "def456...", "", "labelled"],
    ["prev-folder", "9f2c1a7b4d3e5f60"]
  ],
  "content": ""
}
```

### Mark Email as Read and Starred

Two separate events:

**Read event:**
```json
{
  "kind": 1985,
  "tags": [
    ["L", "mail"],
    ["l", "state:read", "mail"],
    ["e", "def456...", "", "labelled"]
  ],
  "content": ""
}
```

**Starred event:**
```json
{
  "kind": 1985,
  "tags": [
    ["L", "mail"],
    ["l", "flag:starred", "mail"],
    ["e", "def456...", "", "labelled"]
  ],
  "content": ""
}
```

### Restore Email from Trash

Publish a deletion request for the gift wrap of the `folder:trash` label event:

```json
{
  "kind": 5,
  "tags": [
    ["e", "<trash_label_gift_wrap_id>"],
    ["k", "1059"]
  ],
  "content": ""
}
```

## Default States

When an email has no associated label events:

| Property | Default State |
|----------|---------------|
| Folder | Natural mailbox |
| Read state | Unread |
| Starred | Not starred |
| Important | Not important |
| Tags | None |
