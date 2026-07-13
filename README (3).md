# Vault — Local Password Manager

A single-file, client-side password manager built for a first-year Cybersecurity capstone. Everything runs in the browser: credentials are encrypted with AES-256-GCM using a key derived from your master password, and the encrypted vault is stored only in the browser's `localStorage`. No backend, no network calls, no plaintext ever written to disk — which makes it a good static site for GitHub Pages.

## Features

- **Master-password-based encryption** — PBKDF2 (SHA-256, 250,000 iterations) derives an AES-256-GCM key from your master password. The master password itself is never stored.
- **Add / edit / delete credentials** — site, username, password, and optional notes per entry.
- **Password generator** — adjustable length, toggleable character sets (lowercase, uppercase, digits, symbols), and an "exclude ambiguous characters" option.
- **Strength meter** — estimates entropy in bits and labels passwords Weak / Fair / Strong / Very strong.
- **Search** — filter entries by site or username.
- **Copy to clipboard** — copies a password and automatically clears the clipboard after 20 seconds.
- **Auto-lock** — vault re-locks after 5 minutes of inactivity.
- **Encrypted export / import** — download a JSON backup of the encrypted vault (salt + IV + ciphertext) and re-import it later or on another device.
- **Erase & start over** — wipes the local vault if you forget the master password.

## How it works

1. On first use, you set a master password. A random 16-byte salt is generated and stored (unencrypted — salts are not secret).
2. Your master password + salt are run through PBKDF2 to derive a 256-bit AES-GCM key. This key exists only in memory while the vault is unlocked; it is never persisted.
3. Every time the vault changes, the full entry list is serialized to JSON, encrypted with AES-256-GCM (fresh random IV each time), and the ciphertext is saved to `localStorage`.
4. Unlocking re-derives the key from the password you enter and attempts to decrypt the stored ciphertext. Wrong password → decryption fails → you're told the password is incorrect (nothing else leaks).
5. Locking the vault (manually or via the idle timer) discards the in-memory key and decrypted data.

## Getting started

### Run locally
Just open `password.html` in any modern browser — no build step, no server required.

### Deploy to GitHub Pages
1. Push `password.html` to a GitHub repository.
2. Rename it to `index.html` if you want it served at your repo's root URL (or leave it as `password.html` and link to it from an existing `index.html`).
3. In the repo settings, go to **Pages** and enable GitHub Pages for the branch/folder containing the file.
4. Visit `https://<your-username>.github.io/<repo-name>/`.

## Usage

- **First visit:** choose a master password (minimum 8 characters — longer and more random is stronger, since it's the only thing protecting the vault).
- **Add an entry:** click **+ New entry**, fill in the site/username/password (or click the generator icon to create one), and save.
- **Copy a password:** click the copy icon next to an entry; it's copied and auto-cleared from the clipboard shortly after.
- **Back up your vault:** click **Export** to download an encrypted JSON file. Keep this somewhere safe — it's still protected by your master password.
- **Restore on another device/browser:** on the lock screen, click **Import vault file**, select your backup, then unlock as normal.

## Tech stack

- Plain HTML/CSS/JavaScript — no frameworks, no build tools, no dependencies to install.
- [Web Crypto API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API) for PBKDF2 key derivation and AES-GCM encryption/decryption.
- Google Fonts (Fraunces, IBM Plex Mono) loaded via CDN.

## Security model & limitations

This project is meant to demonstrate real cryptographic concepts (key derivation, authenticated encryption, salts, IVs) for a capstone — it is **not** a production-grade password manager. Worth knowing before relying on it:

- **Client-side only:** anyone with access to the unlocked browser tab, or to browser dev tools while the vault is unlocked, can read the decrypted data in memory.
- **`localStorage` is not a hardened secret store:** it's convenient for a static-site demo, but it isn't isolated the way an OS keychain or a dedicated secrets manager is.
- **No brute-force throttling:** unlike a real password manager, there's no lockout after repeated failed attempts — the strength of your master password is the only defense.
- **No password recovery:** if you forget the master password, the vault cannot be decrypted. There is no backdoor by design.
- **Single point of trust:** the security of everything depends entirely on the strength and secrecy of your master password.

For a real-world deployment you'd want server-side rate limiting, hardware-backed key storage where available, and independent security review — none of which this project attempts.

## License

Free to use and adapt for coursework or personal projects.
