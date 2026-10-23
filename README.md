# shadow-registry-trust

Public trust material for **Shadow Registry**. It is published on GitHub Pages, outside the main infrastructure, so that it stays reachable even if that infrastructure is compromised or unavailable.

Address: <https://shadow-security-official.github.io/shadow-registry-trust/>

This repository contains no secrets.

## Pinned keys

Shadow Registry clients pin the SPKI fingerprint (SHA-256 of the DER-encoded SubjectPublicKeyInfo) of two certificate authorities.

| Role | Public key | SPKI SHA-256 fingerprint |
|---|---|---|
| Primary Registry CA | `keys/registry-ca-01.pub.pem` | `054062dacf91f33024cfa681fcdf87f02ff26ec4f816769a53b55c991a02c1a2` |
| Backup Registry CA | `keys/registry-ca-backup-01.pub.pem` | `51317763fce23fc57940d841b9c9b8d8511ad05be36bfccc49e530a7ee524698` |

To check a fingerprint:

```bash
openssl pkey -pubin -in keys/registry-ca-01.pub.pem -outform DER | openssl dgst -sha256
```

## Pin revocation

The file `pins/revocation.jws` does not exist under normal conditions: a 404 response means that no revocation has been issued.

Its presence means that the primary Registry CA is permanently revoked. It is a JWS in compact serialization:

- header: `{"alg":"ES384","typ":"shadow-registry-pin-revocation+jws"}`
- payload: `{"format":"shadow-registry-pin-revocation/1","revoked_spki_sha256":"<primary fingerprint>","issued_at":"<date>"}`
- signature: by the backup key, and only by the backup key.

A revocation is valid only if its signature verifies against the backup key and it targets exactly the primary fingerprint. Its origin does not matter: the same file may be published at several addresses.

## Backup authority

After a switchover to the backup authority, this repository also publishes:

- `certs/registry-backup-01.cer`: its self-signed certificate (DER)
- `crl/registry-backup-01.crl`: its certificate revocation list (DER)

These files are served over HTTPS only.

## Primary publication

Under normal conditions, the certificate and revocation list of the primary Registry CA are published at:

- <http://cipher.shadow-security.fr/certs/registry.cer>
- <http://cipher.shadow-security.fr/crl/registry.crl>

## License

The content of this repository is dedicated to the public domain (CC0 1.0, see `LICENSE`), so that anyone can freely copy and republish it.
