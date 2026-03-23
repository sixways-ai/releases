# Feed Signing Keys

The SixWays release feed is signed with Ed25519 using [signify-openbsd](https://man.openbsd.org/signify).

## Public Key

The public key (`feed.pub`) will be placed in this directory once the signing keypair is generated.

## Generating a Keypair

```bash
signify-openbsd -G -n -p feed.pub -s feed.sec
```

- The private key (`feed.sec`) goes into the GitHub Actions secret `FEED_SIGNING_KEY`.
- The public key (`feed.pub`) goes into this directory so consumers can verify signatures.

## Verifying Signatures

Consumers can verify any signed feed file using the public key:

```bash
signify-openbsd -V -p keys/feed.pub -m index.json -x index.json.sig
```

All feed JSON files (index.json, latest.json, channel files, component files, and version files) have corresponding `.sig` files when signing is enabled.
