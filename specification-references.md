# References Between Specifications

A specification can point to another.

```
["a", "30817:<pubkey>:<d>", "<relay hint>", "<marker>"]
["i", "<url>", "<marker>"]
```

The relay hint may be empty. When the source is not a Nostr event, `i` carries its
URL instead, per [NIP-73](nostr:naddr1qvzqqqrcvypzq2eeknl7v2fnm7tsuxfkds3vrcyjl9fls070a465urcy6k3mgk0eqqrxu6ts95mnxm8dm4t).

An unknown marker, or none at all, is a reference and nothing more.

## Fork

Use the marker `fork` to say that this specification was created from another.

## Extends

Use the marker `extends` to say that this specification adds to another.

## Requires

Use the marker `requires` to say that this specification needs another.
