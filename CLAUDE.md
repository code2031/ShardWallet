# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ShardWallet is a Progressive Web App (PWA) wallet for ShardCoin (SHRD). It's a single-page application built with vanilla HTML/CSS/JS — no frameworks, no build step. It connects to a `shardcoind` node via JSON-RPC to display balances, send/receive coins, and show transaction history.

## Running Locally

```bash
python3 -m http.server 8080
```

Open http://localhost:8080. No build step needed.

## Project Structure

```
index.html      — Entire app (HTML structure, CSS styles, JS logic)
manifest.json   — PWA manifest (app name, icons, theme)
sw.js           — Service worker (offline caching, skips RPC requests)
icon-192.png    — App icon 192x192
icon-512.png    — App icon 512x512
```

Everything lives in `index.html`. The app is intentionally a single file for simplicity and portability.

## Architecture

### RPC Layer

The `rpc(method, params)` function sends JSON-RPC POST requests to the configured `shardcoind` node. Config (URL, credentials, wallet name) is stored in `localStorage` under `shardwallet_config`. The RPC interface is Bitcoin-compatible:

- `getbalance` — wallet balance
- `getnewaddress` — generate receive address
- `sendtoaddress` — send coins
- `listtransactions` — transaction history
- `getblockchaininfo` — network/chain status

### Auto-refresh

`refresh()` polls the node every 10 seconds for balance, block info, and transactions.

### Modals

Bottom-sheet style modals for Send, Receive, Network, and Settings. Opened via `openModal(name)`, closed by tapping overlay.

### QR Codes

Uses the qrcode.js library from CDN to render receive addresses as QR codes on a canvas element.

### Service Worker

Caches static assets for offline use. RPC requests (`/rpc`) are excluded from caching and always go to the network.

## ShardCoin RPC Connection

Default mainnet RPC: `http://127.0.0.1:7332`

The node must have `server=1` in `shardcoin.conf` with `rpcuser`/`rpcpassword` set. For cross-origin access from a browser, the node needs CORS headers — typically handled by running a reverse proxy in front of `shardcoind`.

## Deployment

Any static file server works: nginx, Caddy, Apache, GitHub Pages, Netlify, Vercel, or `python3 -m http.server`. HTTPS is required for PWA install and service worker on non-localhost origins.
