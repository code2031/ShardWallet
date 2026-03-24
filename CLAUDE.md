# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ShardWallet is a **non-custodial** Progressive Web App (PWA) wallet for ShardCoin (SHRD). Private keys are generated and stored entirely in the browser — the node is only used for blockchain data and broadcasting transactions. Built with vanilla HTML/CSS/JS and ES module imports from `esm.sh` — no frameworks, no build step.

## Running Locally

```bash
python3 -m http.server 8080
```

Open http://localhost:8080. No build step needed.

## Project Structure

```
index.html      — Entire app (HTML, CSS, JS with ES module imports)
manifest.json   — PWA manifest (app name, icons, theme)
sw.js           — Service worker (offline caching, skips RPC requests)
icon-192.png    — App icon 192x192
icon-512.png    — App icon 512x512
```

Everything lives in `index.html`. The `<script type="module">` block imports crypto libraries from `esm.sh` CDN.

## Architecture

### Key Management (client-side only)

Keys never leave the browser. The crypto stack:

- **BIP39** — 12-word seed phrase generation/validation (`@scure/bip39` from esm.sh)
- **BIP32** — HD key derivation at path `m/84'/1000'/0'/0/*` (`@scure/bip32` from esm.sh)
- **secp256k1** — ECDSA signing (`@noble/curves` from esm.sh)
- **Bech32** — Custom implementation for `shrd` prefix address encoding
- **AES-256-GCM** — Seed phrase encryption in localStorage (Web Crypto API, PBKDF2 200k iterations)

The wallet derives 20 addresses on creation. Addresses use P2WPKH (native segwit, bech32).

### Transaction Signing

Transactions are built and signed entirely in the browser:
1. Fetch UTXOs from node via `listunspent` (watch-only addresses)
2. Build P2WPKH transaction with BIP143 sighash computation
3. Sign with secp256k1 ECDSA using derived private keys
4. Broadcast via `sendrawtransaction` RPC

The `buildSignedTx()` function handles UTXO selection, output construction, BIP143 witness signing, and serialization.

### Node Communication (read-only + broadcast)

The `rpc(method, params)` function sends JSON-RPC POST requests. Node config is stored in `localStorage` under `sw_node`. RPC calls used:

- `getblockchaininfo` — chain status, block height
- `importaddress` — register our addresses as watch-only (no keys sent)
- `listunspent` — get UTXOs for our watch-only addresses
- `listtransactions` — transaction history
- `sendrawtransaction` — broadcast signed transactions

### User Flow

1. **First visit**: Welcome screen → Create wallet (shows seed) or Restore from seed
2. **Password**: Encrypts seed with AES-256-GCM, stores ciphertext in localStorage
3. **Node setup**: Configure RPC connection (or skip for offline mode)
4. **Returning**: Unlock screen → enter password → decrypt seed → derive keys
5. **Lock/Reset**: Available from Settings page

### UI Layout

Monero GUI-inspired design:
- Left sidebar (desktop): branding, balance, navigation, sync status
- Bottom nav (mobile): 5-tab navigation
- Pages: Account (dashboard), Send, Receive, Transactions, Keys & Backup, Advanced, Settings

### ShardCoin Network Parameters

```
Bech32 HRP:     shrd
P2PKH prefix:   63 (addresses start with S)
WIF prefix:     191
BIP44 coin type: 1000
Derivation path: m/84'/1000'/0'/0/*
Mainnet RPC:    http://127.0.0.1:7332
```

## Deployment

Any static file server works: nginx, Caddy, Apache, GitHub Pages, Netlify, Vercel, or `python3 -m http.server`. HTTPS is required for PWA install and service worker on non-localhost origins.

## Security Model

- Private keys exist only in browser memory (cleared on lock) and encrypted in localStorage
- Seed phrase is the sole backup — losing it means losing funds
- Node never receives private keys — only watch-only addresses and pre-signed transactions
- RPC credentials stored in localStorage — use only on trusted devices
- Crypto libraries loaded from esm.sh CDN — verify integrity for production use
