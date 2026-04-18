NIP Trusted Domains
======

Trusted Domains List
------------------------------

`draft` `optional`

## Abstract

Many apps and websites display security warnings (e.g., "Are you sure you want to open this link?") when users click on external links. While this is an important security measure to protect against phishing and malicious websites, it can become a repetitive and frustrating experience for links or domains that the user trusts and visits frequently.

This NIP defines a standard replaceable event that allows users to maintain a synchronized list of trusted domains. Nostr clients can read this list and bypass security warnings when the user clicks on a link that matches an entry in the list.

## Event Definition

This list is a standard list following the NIP-51 specification. The Trusted Links list uses `kind: 10031`. 

Users MAY publish this event to store their trusted domains. 

### Tags

The event uses the following tags to define trusted destinations:

`["domain", "<domain-name>"]`: Specifies an exact trusted domain. Only links where the hostname EXACTLY matches this value SHOULD be considered trusted. Subdomains are NOT automatically trusted.

Example: `["domain", "nostr.build"]` only trusts `nostr.build`. If a user also wants to trust its image CDN, they MUST add a separate `["domain", "image.nostr.build"]` tag.


### Example Event

```json
{
  "kind": 10031,
  "pubkey": "...",
  "created_at": 1678901234,
  "tags": [
    ["domain", "nostr.build"],
    ["domain", "github.com"]
  ],
  "content": "",
  "id": "...",
  "sig": "..."
}
```

## Client Behavior

1. **Fetching**: Clients SHOULD fetch the user's `kind: 10031` event upon login or startup to cache the trusted links list.
2. **Link Handling**: When a user clicks an external link:
   - The client MUST check if the link's hostname EXACTLY matches any `domain` tag in the trusted list.
   - If a match is found, the client SHOULD open the link immediately without displaying a confirmation popup.
   - If no match is found, the client SHOULD display its standard security warning or confirmation dialog.
3. **Adding to the List**: When displaying a security warning for an untrusted link, clients SHOULD provide an option (e.g., a checkbox or button: "Always trust this domain") to add the domain to the user's `kind: 10031` list and publish the updated event.
