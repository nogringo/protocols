# BUD: Blob pinning

`draft` `optional`

A pin is a pubkey's request that a server keep a blob. This BUD lets a client pin a blob for itself or for other pubkeys, pin a blob the server already stores without uploading it, and read its own pin. It adds the `pin` verb to [BUD-11](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cnzuvmwmm).

## PUT /upload and PUT /mirror

The `upload` token ([BUD-11](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cnzuvmwmm)) of `PUT /upload` ([BUD-02](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cryxk74nh)) and `PUT /mirror` ([BUD-04](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95crghgclzs)) MAY include `p` tags, each naming a pubkey to pin the blob for.

- Without `p` tags, the request pins the blob for the signer.
- With `p` tags, it pins the blob for each `p` pubkey, and not for the signer.
- Without a token, it pins the blob for no one.
- If the server refuses one of the `p` pubkeys, it rejects the request with `403 Forbidden`.

`HEAD /upload` ([BUD-06](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95crvcz6ed2)) accepts the same `p` tags and answers as `PUT /upload` would.

```json
{
  "kind": 24242,
  "pubkey": "<one-time pubkey>",
  "created_at": 1772019044,
  "tags": [
    ["t", "upload"],
    ["x", "b1674191a88ec5cdd733e4240a81803105dc412d6c6708d53ab94fc248f4f553"],
    ["p", "<pubkey to pin for>"],
    ["expiration", "1772019344"]
  ],
  "content": "Upload Blob",
  "sig": "<signature>"
}
```

## PUT /pin/sha256 - Pin Blob

Pins a blob the server already stores. The request has no body. The token MUST have `t` set to `pin` and an `x` tag matching `<sha256>`. It MAY include `p` tags, with the same meaning as on upload.

| Status Code        | Meaning                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| `200 OK`           | The blob is pinned. The body contains its [Blob Descriptor](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cryxk74nh#blob-descriptor).           |
| `401 Unauthorized` | Authorization is missing or invalid. See [BUD-11](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cnzuvmwmm#endpoint-authorization-requirements). |
| `403 Forbidden`    | The server refuses one of the `p` pubkeys.                                                      |
| `404 Not Found`    | The server does not store the blob.                                                             |

## GET /pin/sha256 - Get Pin

Returns the signer's own pin on a blob. The token MUST have `t` set to `pin` and an `x` tag matching `<sha256>`.

```json
{
  "sha256": "b1674191a88ec5cdd733e4240a81803105dc412d6c6708d53ab94fc248f4f553",
  "pinned": 1772019044
}
```

`pinned` is the unix timestamp at which the pin was created. Pinning the blob again does not change it. The response never reveals other pins.

| Status Code        | Meaning                                                                                         |
| ------------------ | ----------------------------------------------------------------------------------------------- |
| `200 OK`           | The signer pins the blob. The body describes the pin.                                           |
| `401 Unauthorized` | Authorization is missing or invalid. See [BUD-11](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cnzuvmwmm#endpoint-authorization-requirements). |
| `404 Not Found`    | The signer does not pin the blob.                                                               |

## DELETE /sha256 - Delete Blob

`DELETE /<sha256>` ([BUD-12](nostr:naddr1qvzqqqrcvypzqyek59lpv8gw3tetdrhfttf2g70u8zl0j6sh6cf8agp2grfgmkt7qqrxyaty95cnyqrct8v)) removes the signer's pin and no other, so a pubkey that does not pin a blob cannot delete it. When the last pin is removed, the server MAY delete the blob.
