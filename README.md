# Protocols

New NIPs, Blossom BUDs, a few HTTP APIs, and forks of existing NIPs.

Links between specs are `nostr:naddr` addresses of `kind:30817` spec events. They open in a Nostr client or a spec viewer such as [openspecs](https://openspecs.uid.ovh/npub1kg4sdvz3l4fr99n2jdz2vdxe2mpacva87hkdetv76ywacsfq5leqquw5te), not on GitHub.

## Forks of existing NIPs

| Spec | Change from upstream |
| --- | --- |
| [NIP-07](nip-07.md) | `pubkey` and `current_user` select the user key; machine-readable error prefixes |
| [NIP-37](nip-37.md) | Private Storage relays serve events only to the user who uploaded them |
| [NIP-42](nip-42.md) | `auth-required` vs `restricted` when several pubkeys are authenticated |
| [NIP-46](nip-46.md) | Machine-readable error prefixes |
| [NIP-55](nip-55.md) | Machine-readable error prefixes |
| [NIP-59](nip-59.md) | Gift wrap deletions are hidden from the sender |
| [NIP-9a](nip-9a.md) | Copy of the open draft [#2194](https://github.com/nostr-protocol/nips/pull/2194): relays push events to a callback URL |

## Identity and profiles

| Spec | Summary |
| --- | --- |
| [Fallback Display Name](fallback-display-name.md) | Name to show when kind 0 has none |
| [Default Picture](default-picture.md) | Deterministic avatar color from the pubkey |
| [String Color](string-color.json) | Deterministic UI color from any string (`kind:30818` event) |
| [Profile Themes](profile-themes.md) | Shareable UI themes, the active theme, and an avatar shape |
| [Private Metadata](nip-private-metadata.md) | Profile metadata only its author can read |
| [Alias protocol](manage-nip05.md) | REST API to claim and manage NIP-05 aliases |
| [WKD Publish Protocol](wkd-publish-protocol.md) | Publish OpenPGP keys to a Web Key Directory with NIP-98 |

## Lists

| Spec | Summary |
| --- | --- |
| [Generic Lists](generic-lists.md) | One addressable kind for every list, named by its `d` tag |
| [Append-Only Lists](nip-append-only-lists.md) | One event per list operation, merged as an OR-Set CRDT |
| [Trusted Auth Relays](trusted-auth-relays.md) | Relays the client may AUTH to without asking |
| [Trusted Domains](trusted-domains.md) | Domains whose links skip the external link warning |
| [Address Book](address-book.md) | Private vCard contact cards shared across apps |
| [Mail Contacts](mail-contacts.md) | Mail contacts as an append-only list |

## Relays

| Spec | Summary |
| --- | --- |
| [Computed Home Relays](computed-home-relays.md) | Relays derived from the pubkey, where its `kind:10002` is published |

## Nostr Mail

| Spec | Summary |
| --- | --- |
| [Core](nostr-mail-core.md) | Email over Nostr instead of SMTP, as gift wrapped `kind:1301` |
| [Bridges](nostr-mail-bridge.md) | Gateways to legacy email |
| [Labels](nostr-mail-labels.md) | Folders, read state, stars and tags |
| [Settings](nostr-mail-settings.md) | Settings synced across devices |
| [Large MIME](nostr-mail-large-mime-support.md) | Emails over the NIP-44 size limit, stored encrypted on Blossom |

## Blossom

| Spec | Summary |
| --- | --- |
| [Blob pinning](bud-blob-pinning.md) | Pin a blob for yourself or other pubkeys, and serve it only to them |
| [Inbox servers](bud-inbox-servers.md) | Servers that store the blobs other pubkeys send to a user |

## Scheduling

| Spec | Summary |
| --- | --- |
| [Scheduler DVM](scheduler-dvm.md) | Publish an event at a future time through a DVM, unlinkable by observers |
| [Scheduled Package](scheduled-package.md) | Group several Scheduler DVM jobs into one scheduled item |

## Other

| Spec | Summary |
| --- | --- |
| [Snapshots of Replaceable Events](replaceable-event-snapshots.md) | Keep one version of a replaceable event in a regular event |
| [References Between Specifications](specification-references.md) | `fork`, `extends` and `requires` markers between specs |
