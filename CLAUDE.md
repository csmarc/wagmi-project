# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Next.js 14 application bootstrapped with `create-wagmi`. It demonstrates Web3 wallet connection functionality using wagmi v2, viem, and TanStack Query. The app uses Next.js App Router with Server-Side Rendering (SSR) support for wallet state.

## Development Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linter
npm run lint
```

## Architecture

### Wagmi Configuration (src/wagmi.ts)

The wagmi configuration is centralized in `src/wagmi.ts` using a factory pattern:

- **getConfig()**: Returns a wagmi config with:
  - Chains: Ethereum Mainnet and Sepolia testnet
  - Connectors: Injected wallet, Base Account, and WalletConnect
  - Storage: Cookie-based storage for SSR persistence
  - SSR enabled with `ssr: true`

- **Module augmentation**: Uses TypeScript module augmentation to type the wagmi config globally via the `Register` interface

- **Environment variables**: Requires `NEXT_PUBLIC_WC_PROJECT_ID` for WalletConnect functionality

### Provider Setup

The app uses a two-tier provider pattern for SSR:

1. **Root Layout (src/app/layout.tsx)**:
   - Server component that extracts initial wallet state from cookies using `cookieToInitialState()`
   - Passes initial state to client-side Providers component
   - This enables SSR hydration of wallet connection state

2. **Providers Component (src/app/providers.tsx)**:
   - Client component (marked with `'use client'`)
   - Initializes wagmi config and QueryClient using `useState()` to ensure stable instances
   - Wraps app with WagmiProvider and QueryClientProvider in correct order
   - Receives and forwards `initialState` for SSR hydration

### Page Structure

The main page (src/app/page.tsx) is a client component that demonstrates:
- `useAccount()`: Display connection status, addresses, and chain ID
- `useConnect()`: List available connectors and connect to wallets
- `useDisconnect()`: Disconnect from connected wallet

## Key Patterns

### SSR Cookie Persistence

Wallet state persists across page loads via cookies:
1. wagmi config uses `cookieStorage` in `createStorage()`
2. Root layout reads cookies server-side and converts to initial state
3. Providers component hydrates client with this initial state
4. Changes to wallet state automatically sync back to cookies

### TypeScript Configuration

- Path alias: `@/*` maps to `./src/*`
- Strict mode enabled
- Module resolution: bundler (optimized for Next.js)

## Adding New Chains

To add a new chain:
1. Import chain from `wagmi/chains` in `src/wagmi.ts`
2. Add to `chains` array in `getConfig()`
3. Add transport mapping: `[newChain.id]: http()`

## Adding New Connectors

To add a new connector:
1. Import connector from `wagmi/connectors` in `src/wagmi.ts`
2. Add to `connectors` array in `getConfig()` with required config
