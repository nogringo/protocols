# BUD: Inbox servers

`draft` `optional`

An inbox server stores the blobs other pubkeys send to a user. This BUD defines the inbox server list and how clients use it. It requires [Blob pinning](nostr:naddr1qqgxyaty943xcmmz94cxjmnwd9hxwq3qkg4sdvz3l4fr99n2jdz2vdxe2mpacva87hkdetv76ywacsfq5leqxpqqqpuxz7xega9).

## Inbox server list

A kind `11708` event lists its author's inbox servers, with the `server` tags of [BUD-03](nostr:naddr1qqrxyaty95crxq3qzvm2zlskr58g4u4k3m54454y087r30hedgtavyn75q4yp55dm9lqxpqqqpuxzdv33vk).

```json
{
  "kind": 11708,
  "tags": [
    ["server", "https://inbox.example.com"],
    ["server", "https://inbox.example.net"]
  ],
  "content": ""
}
```

## Sending a blob

A client sends a blob to a pubkey by uploading it to the servers of that pubkey's kind `11708`, with a `p` tag naming the pubkey. When the pubkey has no kind `11708`, the client uses its kind `10063`.

## Receiving a blob

A client looks for a blob sent to it on the servers of its kind `11708`, then on those of its kind `10063`.
