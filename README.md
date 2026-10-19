# Kiwi Wallet

A non-custodial, multi-chain cryptocurrency wallet for Ethereum and Solana, built as a modern single-page web application.

[![Live Demo](https://img.shields.io/badge/demo-kiwiwallet.vercel.app-C8F03C?style=flat-square)](https://kiwiwallet.vercel.app)
![Version](https://img.shields.io/badge/version-3.0.0-10B981?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)
![React](https://img.shields.io/badge/React-18.3-61DAFB?style=flat-square&logo=react&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-5.5-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-5.4-646CFF?style=flat-square&logo=vite&logoColor=white)

**Live application:** https://kiwiwallet.vercel.app

---

## Table of Contents

1. [Overview](#overview)
2. [Features](#features)
3. [Supported Networks](#supported-networks)
4. [Technology Stack](#technology-stack)
5. [Architecture](#architecture)
6. [Project Structure](#project-structure)
7. [Getting Started](#getting-started)
8. [Configuration](#configuration)
9. [Available Scripts](#available-scripts)
10. [Application Walkthrough](#application-walkthrough)
11. [Security Model](#security-model)
12. [Design System](#design-system)
13. [Deployment](#deployment)
14. [Roadmap](#roadmap)
15. [Contributing](#contributing)
16. [License](#license)
17. [Disclaimer](#disclaimer)
18. [Acknowledgments](#acknowledgments)
19. [Contact](#contact)

---

## Overview

Kiwi Wallet is a browser-based wallet that lets users create and manage accounts on both Ethereum and Solana from a single interface. It is non-custodial: wallet keys are generated and held on the user's own device, and the application has no backend that stores or has access to them.

The goal of the project is to provide a clean, approachable alternative to existing browser wallets while keeping the underlying model transparent and open to review. Users can create a wallet from a fresh recovery phrase, hold multiple accounts across both chains simultaneously, switch between mainnet and test networks, send and receive assets, request devnet airdrops, and review their transaction history.

Kiwi is inspired by MetaMask and other modern Web3 wallets, with a focus on a simple user experience, multi-chain support from the start, and a maintainable TypeScript codebase.

## Features

**Wallet management**
- Create a new wallet with a generated 12-word recovery (seed) phrase.
- Derive Ethereum accounts using the `m/44'/60'` HD path via Ethers.js.
- Derive Solana accounts using the `m/44'/501'` HD path via ed25519-hd-key.
- Create both chains simultaneously from one seed phrase in a single step.
- Manage multiple derived accounts within the same wallet session.
- View public and private keys from within the dashboard or setup flow.
- Import wallet from existing seed phrase (work in progress — currently disabled).

**Multi-chain support**
- Ethereum and Solana handled side by side in a single interface.
- Chain-specific logic is isolated so each network uses its native tooling.
- Switch the active token between ETH and SOL within the same dashboard.

**Network switching**
- Toggle between mainnet and test networks.
- Ethereum: Mainnet, Sepolia Testnet, Polygon Mainnet, Arbitrum One, Optimism, Base.
- Solana: Mainnet Beta, Devnet (selectable via Solana-specific RPC configuration).
- Toggle testnet visibility from the network selector panel.

**Transactions**
- Send ETH or SOL through a guided transfer modal.
- Receive assets by displaying the account address.
- Activity tab showing on-chain transaction history for the active address.

**Airdrop (devnet only)**
- Request SOL airdrops on Solana Devnet (0.5, 1, 2, or 2.5 SOL) directly from the dashboard.
- Panel only appears when Devnet is active and SOL is selected.

**Balance**
- Live balance polling every 10 seconds for both ETH and SOL.
- USD value estimates displayed alongside token balances.
- Show/hide balance toggle for privacy.

**Interface**
- Responsive layout for desktop and mobile browsers.
- Dark, terminal-inspired aesthetic with a kiwi-green accent.
- Animated landing page and smooth page transitions via Framer Motion.
- Accessible primitives (Radix UI) for dialogs, menus, dropdowns, and tabs.
- Toast notifications via react-hot-toast and Sonner.
- QR code display for receive addresses via qrcode.react.
- Vercel Analytics and Speed Insights integrated.

**Privacy**
- Private keys stay on the user's device in session state only — nothing is written to `localStorage` or transmitted.
- No third-party analytics beyond Vercel's own infrastructure analytics.

## Supported Networks

| Chain    | Network          | Chain ID | Type    |
|----------|------------------|----------|---------|
| Ethereum | Mainnet          | 1        | Mainnet |
| Ethereum | Sepolia Testnet  | 11155111 | Testnet |
| Polygon  | Polygon Mainnet  | 137      | Mainnet |
| Arbitrum | Arbitrum One     | 42161    | Mainnet |
| Optimism | Optimism         | 10       | Mainnet |
| Base     | Base             | 8453     | Mainnet |
| Solana   | Mainnet Beta     | —        | Mainnet |
| Solana   | Devnet           | —        | Testnet |

Testnet networks are hidden by default in the network selector and can be revealed with the toggle inside the panel.

## Technology Stack

| Area              | Technology                                      |
|-------------------|-------------------------------------------------|
| Framework         | React 18.3                                      |
| Language          | TypeScript 5.5                                  |
| Build tooling     | Vite 5.4                                        |
| Styling           | Tailwind CSS 3.4                                |
| UI primitives     | Radix UI (Alert Dialog, Dialog, Dropdown, Select, Tabs, Toast) |
| Animation         | Framer Motion 11                                |
| Routing           | React Router DOM 6                              |
| Icons             | Lucide React                                    |
| Ethereum          | Ethers.js 6                                     |
| Solana            | @solana/web3.js 1                               |
| Mnemonic / HD     | bip39, ed25519-hd-key, tweetnacl, bs58          |
| Notifications     | react-hot-toast, Sonner                         |
| QR codes          | qrcode.react                                    |
| Theming           | next-themes                                     |
| Analytics         | @vercel/analytics, @vercel/speed-insights       |
| Node polyfills    | vite-plugin-node-polyfills                      |
| Hosting           | Vercel                                          |

## Architecture

Kiwi is a client-only application. All wallet logic runs in the browser, and the only network traffic is JSON-RPC communication with blockchain nodes and Vercel's analytics endpoint.

```
+-----------------------------------------------------------+
|                         Browser                           |
|                                                           |
|   +---------------+     +-----------------------------+   |
|   |   UI Layer    |<--->|     Component-local state   |   |
|   | (React, Radix,|     | accounts, active network,   |   |
|   |  Tailwind,    |     | balances, modal visibility  |   |
|   |  Framer Motion|     |                             |   |
|   +---------------+     +--------------+--------------+   |
|                                        |                  |
|                         +--------------v--------------+   |
|                         |        Chain Scripts        |   |
|                         |  src/components/scripts/    |   |
|                         |  ether.ts  (Ethers.js v6)   |   |
|                         |  solana.ts (@solana/web3.js) |  |
|                         |  sendEth.ts / sendSol.ts    |   |
|                         |  balance.ts / airdropSol.ts |   |
|                         +------+----------------+-----+   |
|                                |                |         |
|                  +-------------v---+   +--------v------+  |
|                  |  Session state  |   |  JSON-RPC     |  |
|                  |  (no persisted  |   |  requests     |  |
|                  |   key storage)  |   |               |  |
|                  +-----------------+   +--------+------+  |
+-------------------------------------------------|---------+
                                                  |
                          +-----------------------v---------+
                          | Infura / Alchemy / public RPC   |
                          | Ethereum and Solana nodes       |
                          +---------------------------------+
```

**Layers**

1. **UI layer.** React components organized by feature, all under `src/components/`. Radix UI supplies accessible primitives, Tailwind CSS handles styling, and Framer Motion handles animation.
2. **Component-local state.** Application state (accounts, active network, balances, modal visibility) is held locally in component `useState` hooks and passed via props. There is no separate React Context or Redux store.
3. **Chain scripts.** Ethereum and Solana each have dedicated script files under `src/components/scripts/`. These wrap the native SDKs and are called directly from component handlers.
4. **Storage.** Keys live in React component state for the duration of the session only. Nothing is written to `localStorage`, `sessionStorage`, or any server.
5. **Network layer.** Balances, fees, and transaction submission happen over JSON-RPC to the configured provider for the selected network.

## Project Structure

<!-- Keep this tree in sync with the repository when directories change. -->

```
kiwi/
├── public/
│   ├── kiwi-icon.svg
│   └── LogoWallets/            Chain and network icon assets
├── src/
│   ├── components/
│   │   ├── ui/                 Radix-based primitives (button, dialog, input,
│   │   │                       tabs, select, toast, badge, card, alert, etc.)
│   │   ├── landing/
│   │   │   ├── Globe.tsx       Animated globe graphic for landing page
│   │   │   └── graphics.tsx    GridFloor and other decorative SVG elements
│   │   ├── scripts/
│   │   │   ├── ether.ts        HD wallet derivation for Ethereum (Ethers.js v6)
│   │   │   ├── solana.ts       HD wallet derivation for Solana + mnemonic gen
│   │   │   ├── balance.ts      ETH and SOL balance fetching
│   │   │   ├── sendEth.ts      Ethereum transfer logic
│   │   │   ├── sendSol.ts      Solana transfer logic
│   │   │   └── airdropSol.ts   Solana Devnet airdrop request
│   │   ├── Landing.tsx         Landing page (hero, features, CTA)
│   │   ├── Dashboard.tsx       Main wallet dashboard
│   │   ├── SetupWallet.tsx     Wallet creation and account derivation flow
│   │   ├── NetworkSelector.tsx Network switching dropdown
│   │   ├── ActivityList.tsx    On-chain transaction history tab
│   │   ├── AirdropPanel.tsx    Devnet SOL airdrop panel
│   │   ├── SendCrypto.tsx      Send transaction modal
│   │   ├── ReceiveCrypto.tsx   Receive address + QR code modal
│   │   ├── SettingsModal.tsx   Settings overlay
│   │   ├── ShowKeys.tsx        Key viewer dialog (public + private)
│   │   ├── AccountPicker.tsx   Account switching UI
│   │   ├── Notification.tsx    Single notification component
│   │   ├── NotificationContainer.tsx  Notification stack renderer
│   │   ├── brand.tsx           Shared brand elements (TopBar, HatchStrip, KiwiMark)
│   │   └── 404.tsx             Not-found page
│   ├── hooks/
│   │   ├── useWalletBalance.ts Balance polling hook (10 s interval, multi-wallet aware)
│   │   └── useNotifications.ts Notification queue hook
│   ├── lib/
│   │   └── utils.ts            cn() class-name helper (clsx + tailwind-merge)
│   ├── assets/                 Static assets imported by components
│   ├── App.tsx                 Router and route definitions
│   ├── main.tsx                Application entry point
│   ├── index.css               Tailwind directives and global styles
│   └── vite-env.d.ts           Vite env type declarations
├── .env                        Local environment variables (not committed)
├── index.html                  HTML entry and metadata
├── package.json
├── tailwind.config.js          Theme, design tokens, and custom animations
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── vite.config.ts
├── vercel.json                 SPA rewrites + security headers
└── README.md
```

### Key Components

| Component               | Responsibility                                                                          |
|-------------------------|-----------------------------------------------------------------------------------------|
| `Landing.tsx`           | Introduces the product with a hero section, animated globe, and chain graphics.        |
| `SetupWallet.tsx`       | Guides the user through naming, chain selection, seed phrase backup, and key derivation.|
| `Dashboard.tsx`         | Main interface: balance display, asset list, activity tab, and action buttons.          |
| `NetworkSelector.tsx`   | Dropdown that lists all supported EVM networks; togglable testnet visibility.           |
| `AirdropPanel.tsx`      | Requests SOL from the Devnet faucet; only visible on Devnet with SOL selected.          |
| `SendCrypto.tsx`        | Modal flow for signing and submitting ETH or SOL transfers.                             |
| `ReceiveCrypto.tsx`     | Displays the active public address with a QR code for incoming funds.                  |
| `ActivityList.tsx`      | Fetches and displays on-chain transaction history for the active address.               |
| `ShowKeys.tsx`          | Reveals the public and private key pair in a confirmation-gated dialog.                 |
| `brand.tsx`             | Shared layout chrome: `TopBar`, `HatchStrip`, `KiwiMark`, `FieldLabel`.                |
| `useWalletBalance.ts`   | Polls ETH and SOL balances every 10 seconds; handles single and multi-wallet modes.     |
| `useNotifications.ts`   | Manages a queue of timed toast notifications across the dashboard.                      |

## Getting Started

### Prerequisites

- Node.js 18 or later
- npm
- Git

### Installation

```bash
# Clone the repository
git clone https://github.com/SheerWill007/kiwi-wallet.git
cd kiwi-wallet

# Install dependencies
npm install

# Create the environment file (see Configuration)
cp .env.example .env

# Start the development server
npm run dev
```

The application will be available at `http://localhost:5173`.

## Configuration

Create a `.env` file in the project root. The `.env.example` file provides the full template:

```env
# Application
VITE_APP_NAME=Kiwi Wallet
VITE_APP_VERSION=3.0.0

# Network Configuration
VITE_SOLANA_MAINNET_RPC=https://api.mainnet-beta.solana.com
VITE_SOLANA_DEVNET_RPC=https://api.devnet.solana.com
VITE_ETHEREUM_MAINNET_RPC=https://eth-mainnet.g.alchemy.com/v2/YOUR_KEY_HERE
VITE_ETHEREUM_TESTNET_RPC=https://eth-sepolia.g.alchemy.com/v2/YOUR_KEY_HERE

# Feature Flags
VITE_ENABLE_TESTNET=true
VITE_ENABLE_AIRDROP=true
```

| Variable                     | Description                                                    |
|------------------------------|----------------------------------------------------------------|
| `VITE_APP_NAME`              | Application display name.                                      |
| `VITE_APP_VERSION`           | Application version string.                                    |
| `VITE_SOLANA_MAINNET_RPC`    | Solana Mainnet Beta RPC endpoint.                              |
| `VITE_SOLANA_DEVNET_RPC`     | Solana Devnet RPC endpoint.                                    |
| `VITE_ETHEREUM_MAINNET_RPC`  | Ethereum Mainnet RPC endpoint (Alchemy, Infura, or public).    |
| `VITE_ETHEREUM_TESTNET_RPC`  | Ethereum Sepolia RPC endpoint.                                 |
| `VITE_ENABLE_TESTNET`        | Feature flag to show testnet networks.                         |
| `VITE_ENABLE_AIRDROP`        | Feature flag to enable the devnet airdrop panel.               |

Because Vite embeds `VITE_` variables into the client bundle, these values are visible to anyone who loads the application. Restrict RPC keys using the allowlist and rate-limit options provided by your RPC provider, and never place secrets in these variables.

## Available Scripts

| Command              | Description                                          |
|----------------------|------------------------------------------------------|
| `npm run dev`        | Start the development server with hot reload.        |
| `npm run build`      | Type-check and create an optimized production build. |
| `npm run preview`    | Serve the production build locally for verification. |
| `npm run lint`       | Run ESLint across the project.                       |
| `npm run type-check` | Run TypeScript type checking without emitting files. |

## Application Walkthrough

1. **Landing.** The user arrives at the landing page with an animated globe and chain graphics and chooses to get started.
2. **Wallet setup.** The user names their wallet, then selects a path:
   - **Create wallet** — picks a chain (ETH, SOL, or both), is shown a generated 12-word recovery phrase to back up, and accounts are derived using BIP-44 HD paths.
   - **Import wallet** — currently a work in progress and disabled.
3. **Dashboard.** After setup, the dashboard shows the active account name, address, balance, and USD estimate. An assets tab lists all tokens; an activity tab shows transaction history.
4. **Network selection.** The network selector dropdown lists Ethereum Mainnet, Sepolia, Polygon, Arbitrum, Optimism, and Base. Testnet networks can be toggled on/off in the panel.
5. **Devnet airdrop.** When Solana Devnet is active and SOL is the selected currency, an airdrop panel appears to request test SOL from the faucet.
6. **Send and receive.** The Send modal builds and submits a transfer on the active network. The Receive modal displays the account address with a QR code.
7. **Keys.** The `.keys [show]` button in the top bar opens a dialog revealing the public and private key for the active account.

## Security Model

Kiwi follows a non-custodial design.

- **Session-only key storage.** Private keys and recovery phrases are generated and held only in React component state for the lifetime of the browser session. They are never written to `localStorage`, `sessionStorage`, cookies, or any server.
- **No backend.** The application is a fully static build. There is no Kiwi-operated server that receives or can access wallet data.
- **Recovery phrase.** Wallets are recoverable through a 12-word BIP-39 mnemonic. Whoever holds this phrase controls the funds.
- **Security headers.** The Vercel deployment enforces `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, and `X-XSS-Protection: 1; mode=block` on all responses.
- **Open source.** The codebase is public so that the community can inspect and review it.

### Limitations and recommendations

- Because keys exist only in session state, refreshing or closing the tab will clear them. You will need your recovery phrase to restore access.
- Browser-based wallets are exposed to the risks of the browser environment, including malicious extensions, compromised devices, and phishing sites. Only use the application from the official URL listed above.
- The project has not undergone a formal third-party security audit. Do not use it to hold funds you cannot afford to lose.
- Use test networks (Sepolia, Devnet) while evaluating the application.
- Store the recovery phrase offline. Never share it, and never enter it on any site other than a wallet you trust.

To report a vulnerability, please open a private security advisory on the repository rather than a public issue.

## Design System

The interface uses a terminal-inspired aesthetic with a signature kiwi-green accent derived from the kiwi fruit.

| Role        | Name          | Hex / Value           |
|-------------|---------------|-----------------------|
| Primary     | Kiwi          | `#C8F03C`             |
| Kiwi dim    | Kiwi Dim      | `#9DBF22`             |
| Kiwi glow   | Kiwi Glow     | `rgba(200,240,60,.35)`|
| Background  | Ink           | `#000000`             |
| Panel       | Panel         | `#0a0a0a`             |
| Line        | Line          | `rgba(255,255,255,.14)`|
| Line strong | Line Strong   | `rgba(255,255,255,.32)`|

**Typography**

| Role    | Family                          |
|---------|---------------------------------|
| Display | Space Grotesk                   |
| Mono    | JetBrains Mono                  |

Tailwind CSS provides the utility-first styling layer. Radix UI primitives supply accessible interactive components. Framer Motion drives page transitions and micro-animations. Custom keyframe animations include `marquee`, `float`, and `blink` (used for the terminal cursor effect).

## Deployment

The application is a static Vite build and deploys to any static hosting provider. The production instance runs on Vercel.

```bash
npm run build
```

The output is written to the `dist/` directory. To deploy on Vercel, import the repository, set the framework preset to Vite, and define the environment variables listed in the Configuration section in the project settings.

The `vercel.json` file configures:
- SPA rewrites (all routes → `index.html`) so React Router handles client-side navigation.
- Security headers (`X-Content-Type-Options`, `X-Frame-Options`, `X-XSS-Protection`) on all routes.
- Long-term cache headers (`Cache-Control: public, max-age=31536000, immutable`) on hashed static assets under `/assets/`.

## Roadmap

Planned or potential areas of improvement:

- Import wallet from existing seed phrase (currently disabled / work in progress)
- Token and NFT support beyond native ETH and SOL assets
- Persistent encrypted key storage (with user-set password) so sessions survive page reloads
- Hardware wallet integration
- Transaction simulation and fee estimation improvements
- Additional EVM-compatible networks and custom network support
- Automated test coverage and a third-party security audit

## Contributing

Contributions are welcome.

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Commit your changes: `git commit -m "Add your feature"`
4. Push the branch: `git push origin feature/your-feature`
5. Open a pull request describing the change and its motivation.

Please keep changes focused, follow the existing TypeScript and formatting conventions, and include a clear description of any security-relevant behavior.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Disclaimer

Kiwi Wallet is provided "as is", without warranty of any kind. Use at your own risk. The authors are not responsible for any loss of funds. Always keep your seed phrase secure and never share it with anyone.

## Acknowledgments

- Inspired by MetaMask and other modern Web3 wallets.
- Built by Amcha.
- Thanks to the Ethereum and Solana communities and the maintainers of the open source libraries this project depends on.

## Contact

- Portfolio: [WillxTech](https://willx.tech)
- GitHub: [SheerWill007](https://github.com/SheerWill007)
