# Authentication (Multi-Provider Sign-In)

## Table of Contents

- [Authentication (Multi-Provider Sign-In)](#authentication-multi-provider-sign-in)
  - [Table of Contents](#table-of-contents)
  - [Overview](#overview)
  - [How It Works](#how-it-works)
  - [Supported Providers](#supported-providers)
  - [Email Authentication](#email-authentication)
    - [Setup](#setup)
    - [Email Flow](#email-flow)
  - [Social Login](#social-login)
    - [Supported Providers](#supported-providers-1)
    - [Implementation](#implementation)
  - [Passkeys](#passkeys)
    - [Overview](#overview-1)
    - [Implementation](#implementation-1)
  - [Traditional Wallets](#traditional-wallets)
    - [MetaMask / WalletConnect](#metamask--walletconnect)
  - [Backend Verification](#backend-verification)
    - [Verify Signature](#verify-signature)
    - [Full Backend Example](#full-backend-example)
  - [Framework Integration](#framework-integration)
    - [With Wagmi](#with-wagmi)
    - [With Viem](#with-viem)

## Overview

GateLayer AA Wallet supports multiple authentication providers through Alchemy AA SDK. Users can sign in with Email, Social accounts, Passkeys, or traditional wallets - all accessing the same Light Account smart contract.

## How It Works

1. User selects authentication method
2. Provider returns a signer (owner) for the Light Account
3. Light Account address is derived from owner
4. Signer is used to sign all transactions
5. Smart contract (Light Account) verifies signatures via ERC-1271

## Supported Providers

| Provider | Type | Setup Complexity | User Experience |
|----------|------|------------------|-----------------|
| Email | Passwordless | Medium | Web2-familiar |
| Google/Facebook | OAuth | Medium | Social login |
| Passkeys | WebAuthn | Low | Biometric |
| MetaMask | EOA Wallet | Low | Web3 familiar |
| WalletConnect | Mobile Wallet | Low | Mobile-first |
| Privy | MPC | High | Embedded wallet |

## Email Authentication

### Setup

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";

// Use Magic.link-style authentication
const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerChain,
  // Alchemy handles email authentication internally
});
```

### Email Flow

```typescript
// 1. Request authentication email
await client.authenticate({
  type: "email",
  email: user@example.com
});

// 2. User clicks link in email (verified by Alchemy)

// 3. Access granted with signer
const address = client.account.address;
```

## Social Login

### Supported Providers

- Google
- Apple
- Facebook
- Discord

### Implementation

```typescript
// OAuth flow returns Alchemy token
const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerChain,
  authToken: socialOAuthToken,
});
```

## Passkeys

### Overview

Passkeys use WebAuthn for biometric authentication (FaceID, TouchID, Windows Hello).

### Implementation

```typescript
const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerChain,
  passkey: true,
});

// Triggers browser passkey prompt
await client.signInWithPasskey();
```

## Traditional Wallets

### MetaMask / WalletConnect

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";
import { walletClient } from "./wallet-setup";

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerChain,
  owner: walletClient, // EOA signer
});
```

## Backend Verification

### Verify Signature

```typescript
import { createPublicClient, http } from "viem";
import { gatelayerMainnet } from "./chain";

const publicClient = createPublicClient({
  chain: gatelayerMainnet,
  transport: http(),
});

// Verify message signature (handles ERC-6492 for undeployed accounts)
const isValid = await publicClient.verifyMessage({
  address: accountAddress,
  message: signInMessage,
  signature: signature,
});

if (!isValid) {
  throw new Error("Invalid signature");
}
```

### Full Backend Example

```typescript
import express from "express";

const app = express();
const usedNonces = new Set<string>();

// Generate nonce
app.get("/auth/nonce", (req, res) => {
  const nonce = crypto.randomUUID().replace(/-/g, "");
  res.json({ nonce });
});

// Verify authentication
app.post("/auth/verify", async (req, res) => {
  const { address, message, signature } = req.body;

  // Extract nonce from message
  const nonceMatch = message.match(/Nonce: (\w+)/);
  if (!nonceMatch || usedNonces.has(nonceMatch[1])) {
    return res.status(401).json({ error: "Invalid or reused nonce" });
  }

  // Verify signature
  const isValid = await publicClient.verifyMessage({
    address,
    message,
    signature,
  });

  if (!isValid) {
    return res.status(401).json({ error: "Invalid signature" });
  }

  // Mark nonce as used
  usedNonces.add(nonceMatch[1]);

  // Create session
  req.session.address = address;
  res.json({ success: true, address });
});
```

## Framework Integration

### With Wagmi

```typescript
import { createConfig } from "wagmi";
import { gatelayerMainnet } from "./chain";
import { custom } from "viem";

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
});

const config = createConfig({
  chains: [gatelayerMainnet],
  connectors: [
    custom({
      provider: client.getProvider(),
    }),
  ],
});
```

### With Viem

```typescript
import { createWalletClient, custom } from "viem";

const walletClient = createWalletClient({
  chain: gatelayerMainnet,
  transport: custom(client.getProvider()),
});
```

