# Vault Key security model

Vault Key is a fully client-side browser vault. It has no account, backend, sync service, analytics, external assets, or telemetry.

## Cryptography

- KDF: Web Crypto PBKDF2-HMAC-SHA-256, 650,000 iterations, with a random 128-bit salt per vault.
- Encryption: AES-256-GCM. Metadata and every entry are encrypted separately with a fresh 96-bit IV from `crypto.getRandomValues` on every save.
- Randomness: salts, IVs, generated passwords, and entry IDs use Web Crypto. `Math.random` is never used.
- Storage: localStorage contains format/KDF/cipher metadata, salt, IVs, and ciphertext. Titles, usernames, passwords, URLs, notes, and settings are encrypted.
- Authentication: AES-GCM authentication rejects a wrong passphrase or modified vault.
- Clipboard: copied passwords are cleared after about 30 seconds when browser clipboard permissions allow reading the current clipboard. Browsers can deny clipboard reads, so clearing cannot be guaranteed.
- Export: an export is the encrypted vault envelope as stored. Import validates decryption before replacing the current vault.

PBKDF2 was chosen over Argon2id to avoid a WASM dependency and keep the app small, offline, and auditable. PBKDF2 is CPU-hard, not memory-hard. 650,000 iterations raises offline guessing cost, but a weak passphrase can still be guessed. Use a long, unique passphrase.

## What it protects against

A copied localStorage database or exported file does not reveal vault fields without the passphrase, assuming browser cryptography and the passphrase are sound. Locking drops the in-memory key and decrypted data, clears rendered entry data and sensitive form values, and reload always starts locked.

## What it does not protect against

- A compromised device, browser, extension, operating system, or page context can read entries while unlocked, intercept the passphrase, or capture the screen or keyboard.
- XSS or a malicious change to the shipped HTML can access unlocked secrets. The Content Security Policy blocks network connections and third-party resources, but a web app still has an XSS surface.
- There is no browser-extension autofill or phishing-origin binding. Users must verify where they paste credentials.
- Clipboard managers and the operating system may retain copied data despite the best-effort clear.
- localStorage can be erased by browser cleanup, private mode, quota eviction, or profile loss.
- There is no cloud backup or sync. Make and safely store encrypted exports if backup matters.
- There is no recovery, reset, escrow, recovery key, or hidden access. Losing the passphrase or the only stored vault means permanent data loss.
- Metadata such as vault existence, record count, KDF settings, ciphertext sizes, and modification activity may be visible.

## Deployment and audit

The app is a single static HTML file plus documentation. Its CSP sets `connect-src 'none'`; normal operation makes no requests after the document loads. GitHub Pages necessarily serves the initial files. Audit the deployed source and browser Network panel when evaluating a build.
