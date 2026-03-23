ShardWallet
=====================================

A Progressive Web App (PWA) wallet for [ShardCoin](https://github.com/code2031/ShardCoin) (SHRD).

Installable on any device — phones, tablets, desktops — directly from the browser. No app store required.

### Features

- **Balance dashboard** with live updates from your ShardCoin node
- **Send SHRD** to any address with amount validation
- **Receive SHRD** with QR code generation and one-tap copy
- **Transaction history** with send/receive/mined categorization
- **Network stats** — block height, difficulty, chain, disk usage
- **Configurable RPC** — connect to any ShardCoin node
- **Offline-capable** via service worker caching
- **Installable** as a native-like app on any platform

### Screenshots

Dark theme with green accent. Mobile-first responsive layout with bottom navigation.

Setup
-----

ShardWallet connects to a running `shardcoind` node via JSON-RPC.

### 1. Start your ShardCoin node

```bash
# Enable RPC in shardcoin.conf
server=1
rpcuser=myuser
rpcpassword=mypass
rpcallowip=127.0.0.1

# For remote wallet access, also add:
rpcallowip=0.0.0.0/0
rpcbind=0.0.0.0
```

```bash
shardcoind -daemon
```

### 2. Serve the wallet

```bash
cd ShardWallet
python3 -m http.server 8080
```

Or use any static file server (nginx, Apache, Caddy, etc.).

### 3. Open and configure

Open `http://localhost:8080` in your browser.

Tap the gear icon and enter your RPC connection details:
- **RPC URL**: `http://127.0.0.1:7332`
- **RPC Username**: your `rpcuser`
- **RPC Password**: your `rpcpassword`
- **Wallet Name**: (optional, leave blank for default)

### 4. Install as app

- **iOS**: Safari → Share → "Add to Home Screen"
- **Android**: Chrome → Menu → "Install app"
- **Desktop**: Chrome/Edge → address bar install icon

Network Ports
-------------

| Network | RPC Port |
|---------|----------|
| Mainnet | 7332 |
| Testnet | 17332 |
| Regtest | 17443 |

Tech Stack
----------

- Pure HTML / CSS / JavaScript — no frameworks, no build step
- Service worker for offline caching
- [qrcode.js](https://github.com/soldair/node-qrcode) for QR generation
- Web Manifest for PWA install

Security Notes
--------------

- RPC credentials are stored in browser `localStorage` — only use on trusted devices
- For remote access, always run behind HTTPS (use a reverse proxy like Caddy or nginx)
- The wallet does not hold private keys — those live in `shardcoind`

License
-------

MIT
