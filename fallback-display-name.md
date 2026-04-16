Fallback Display Name
---------------------

`draft` `optional`

When a user has no `name` or `display_name` in their kind 0 metadata, clients SHOULD display:

1. The local part of `nip05` (the part before `@`) if available and non-empty
2. `anon_<first 8 characters after "npub1">` if no nip05 is present

### Examples

| npub | nip05 | Fallback name |
|------|-------|---------------|
| `npub1qrzc3xegzwtc7jnx5snq2hr8j34ph2lp3gz3pqu549d2g04pv0pqa4n5g0` | `alice@example.com` | `alice` |
| `npub1sg6plzptd64u62a878hep2kev88swjh3tw00gjsfl8f237lmu63q0uf63m` | (none) | `anon_sg6plzpt` |
| `npub1xyklvwq3s0e5v7r0v8w9x0y1z2a3b4c5d6e7f8g9h0i1j2k3l4m5n6o7p8q9r0s` | `bob@nostr.com` | `bob` |

### Rules

- **nip05 fallback**: Extract the local part (before `@`) from the nip05 identifier
- **anon fallback**: Prefix `anon_` + exactly 8 characters after `npub1`

### Display Name Resolution Order

1. `display_name` from kind 0
2. `name` from kind 0
3. Local part of `nip05`
4. `anon_xxxxxxxx` as defined above
