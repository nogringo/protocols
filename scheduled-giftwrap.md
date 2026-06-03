# Scheduled Giftwrap Protocol

`draft` `optional`

This document describes how a client application can schedule giftwrap events (kind:1059) for future delivery using a Scheduler DVM, while preserving the ability to list, display, and cancel scheduled messages from any device.

## Overview

Scheduling a giftwrap requires two complementary events published by the client:

1. A **kind:5905** job request sent to the Scheduler DVM (see Scheduler DVM NIP)
2. A **kind:31234** draft event (NIP-37) stored on relays, containing the encrypted rumor, linked to the job request

The draft event allows any device owned by the user to reconstruct the full message content and cancel the schedule if needed.

## 1. Publishing a Scheduled Message

When the user schedules a message, the client performs two operations in parallel:

### 1a. Scheduler DVM job request (kind:5905)

See the Scheduler DVM NIP. The encrypted payload contains the fully constructed giftwrap (kind:1059) as the `signed_event`.

### 1b. Draft event (kind:31234)

The client publishes a NIP-37 draft event containing the rumor encrypted with the user's own key. The `e` tag links the draft to its corresponding DVM job request:

```json
{
  "kind": 31234,
  "pubkey": "<client_pubkey>",
  "content": "<nip44_encrypt(client_privkey, client_pubkey, JSON.stringify(rumor))>",
  "tags": [
    ["d", "<random_id>"],
    ["k", "1059"],
    ["e", "<kind5905_event_id>"],
  ]
}
```

| Tag | Description |
|-----|-------------|
| `d` | A random identifier, unique per draft |
| `k` | The kind of the drafted event - always `1059` for giftwraps |
| `e` | The event ID of the corresponding kind:5905 job request |

The `content` is the rumor JSON-stringified and NIP-44 encrypted to the user's own key pair. The rumor contains all information needed to display the message: recipient, subject, body, and any other application-level fields.

Clients SHOULD publish kind:31234 events to the relays listed in the user's kind:10013 event. Per NIP-37, those relay URLs are themselves NIP-44 encrypted inside the kind:10013 content.

## 2. Listing Scheduled Messages

The client first decrypts its kind:10013 content to retrieve its private relay list, then fetches from those relays:

```json
{
  "authors": ["<client_pubkey>"],
  "kinds": [31234],
  "#k": ["1059"]
}
```

For each event, the client:

1. Decrypts `content` using its own key pair to recover the rumor
2. Reads the `e` tag to retrieve the associated kind:5905 event ID
3. Displays the message using the rumor's fields (recipient, subject, body, etc.)

## 3. Cancelling a Scheduled Message

To cancel a schedule, the client publishes two events:

**Cancel the DVM job:**
```json
{
  "kind": 5,
  "tags": [
    ["e", "<kind5905_event_id>"]
  ]
}
```

**Delete the draft** (per NIP-37, blank the content):
```json
{
  "kind": 31234,
  "content": "",
  "tags": [
    ["d", "<same_random_id>"],
    ["k", "1059"]
  ]
}
```
