# GateLayer AA Wallet Skill Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Create a comprehensive Agent Skill for integrating Alchemy Account Abstraction wallets on GateLayer network, enabling AI agents to help developers implement smart wallet functionality.

**Architecture:** Single skill directory with main SKILL.md file and five reference documents. Structure mirrors Base Account skill for AI agent familiarity. Uses Alchemy AA SDK (Light Account) configured for GateLayer Mainnet (Chain ID: 10088) and Testnet (Chain ID: 10087). References existing `connecting-to-gatelayer-network` skill for network configuration.

**Tech Stack:** Alchemy AA SDK, TypeScript/JavaScript, ERC-4337, Markdown documentation

---

## Task 1: Create Directory Structure

**Files:**
- Create: `skills/building-with-gatelayer-aa/`
- Create: `skills/building-with-gatelayer-aa/references/`

**Step 1: Create main skill directory**

Run:
```bash
mkdir -p skills/building-with-gatelayer-aa/references
```

Expected: Directory created successfully

**Step 2: Verify directory structure**

Run:
```bash
ls -la skills/building-with-gatelayer-aa/
```

Expected: Output shows `references/` subdirectory

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/
git commit -m "feat: create GateLayer AA Wallet skill directory structure"
```

---

## Task 2: Write Main SKILL.md - Front Matter and Overview

**Files:**
- Create: `skills/building-with-gatelayer-aa/SKILL.md`

**Step 1: Write SKILL.md front matter and introduction**

Create file with:

```markdown
---
name: building-with-gatelayer-aa
description: Integrates Alchemy AA SDK for GateLayer smart wallet functionality. Covers Light Account creation, multi-provider authentication (Email, Social, Passkey), gas sponsorship via Paymasters, and batch transactions. Use when building apps with smart wallet authentication, sponsored transactions, or batched operations on GateLayer. Covers phrases like "create AA wallet on GateLayer", "GateLayer smart wallet", "Gasless transactions GateLayer", "Paymaster setup", "batch transactions GateLayer", or "account abstraction GateLayer".
---

# Building with GateLayer AA Wallet

GateLayer AA Wallet provides ERC-4337 smart wallet functionality using Alchemy's Light Account. Enable passwordless authentication, gasless transactions, and batched operations on GateLayer Mainnet and Testnet.
```

**Step 2: Run validation**

Run:
```bash
cat skills/building-with-gatelayer-aa/SKILL.md | head -20
```

Expected: Front matter YAML and title displayed correctly

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: add SKILL.md front matter and overview"
```

---

## Task 3: Write SKILL.md - Quick Start Section

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md`

**Step 1: Add Quick Start section**

Append to SKILL.md:

```markdown
## Quick Start

### Installation

```bash
npm install @alchemy/aa-alchemy @alchemy/aa-accounts @alchemy/aa-core viem
```

### Basic Setup

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";
import { http } from "viem";

// GateLayer network configuration
const gatelayerMainnet = {
  id: 10088, // From connecting-to-gatelayer-network skill
  name: "GateLayer",
  rpcUrls: {
    public: {
      http: ["https://gatelayer-mainnet.gatenode.cc"]
    }
  },
  nativeCurrency: {
    name: "GT",
    symbol: "GT",
    decimals: 18
  }
};

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
  owner: signer, // Obtained from authentication
});

// Get account address
console.log("Smart Account Address:", client.account.address);
```

### First Transaction

```typescript
const result = await client.sendUserOperation({
  uo: {
    target: "0xRecipientAddress",
    data: "0x",
    value: 0n,
  },
});

console.log("Transaction Hash:", result.hash);
```
```

**Step 2: Verify markdown syntax**

Run:
```bash
# Check for basic markdown errors
grep -n '```' skills/building-with-gatelayer-aa/SKILL.md | wc -l
```

Expected: Even number (code blocks properly closed)

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: add Quick Start section to SKILL.md"
```

---

## Task 4: Write SKILL.md - Feature Reference Table

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md`

**Step 1: Add feature reference table**

Append to SKILL.md:

```markdown
## Feature References

Read the reference for the feature you're implementing:

| Feature | Reference | When to Read |
|---------|-----------|-------------|
| Authentication | [references/authentication.md](references/authentication.md) | Email/Social/Passkey login, backend verification, multi-provider setup |
| Smart Wallet | [references/smart-wallet.md](references/smart-wallet.md) | Light Account creation, import, management, signing |
| Paymaster | [references/paymaster.md](references/paymaster.md) | Gas sponsorship setup, policy configuration, cost control |
| Batch Transactions | [references/batch-transactions.md](references/batch-transactions.md) | Atomic batching, multi-call execution, gas optimization |
| Troubleshooting | [references/troubleshooting.md](references/troubleshooting.md) | Debugging, common errors, FAQ, best practices |
```

**Step 2: Validate table markdown**

Run:
```bash
# Check table syntax
head -30 skills/building-with-gatelayer-aa/SKILL.md | tail -15
```

Expected: Table renders correctly with pipe separators

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: add feature reference table to SKILL.md"
```

---

## Task 5: Write SKILL.md - Critical Requirements Section

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md`

**Step 1: Add Critical Requirements section**

Append to SKILL.md:

```markdown
## Critical Requirements

### Security

- **Track transaction IDs** to prevent replay attacks - use database with unique constraints
- **Verify sender matches authenticated user** to prevent impersonation
- **Never expose Paymaster URLs** client-side - use a proxy server
- **Paymaster providers must be ERC-7677 compliant** with Alchemy AA SDK
- **Generate nonces before user interaction** to avoid popup blockers

### Network Configuration

- **Use connecting-to-gatelayer-network skill** to get correct Chain IDs and RPC endpoints
- **Mainnet Chain ID**: 10088
- **Testnet Chain ID**: 10087
- **Always validate Chain ID** before signing transactions
- **Use HTTPS RPC endpoints only** - reject any `http://` endpoints

### Popup Handling

- Generate nonces **before** user clicks "Sign in" button
- Use `Cross-Origin-Opener-Policy: same-origin-allow-popups`
- **Do NOT use** `same-origin` - it breaks wallet popups
- Handle popup rejections gracefully with user feedback

### Light Account Specific

- **ERC-6492 wrapper** enables signature verification before deployment
- Viem's `verifyMessage`/`verifyTypedData` handle this automatically
- Account may not be deployed until first transaction
- Check deployment status with `client.isAccountDeployed()`

### Gas Estimation

- Always estimate gas before sending transactions
- Handle paymaster failures gracefully
- Provide fallback to user-paid gas when sponsorship unavailable
- Consider auxiliary funds (off-chain balances) when showing "insufficient funds"
```

**Step 2: Verify section headers**

Run:
```bash
grep -n "^###" skills/building-with-gatelayer-aa/SKILL.md | tail -10
```

Expected: All subsection headers properly formatted

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: add Critical Requirements section"
```

---

## Task 6: Write SKILL.md - Architecture and Final Sections

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md`

**Step 1: Add Architecture and Links sections**

Append to SKILL.md:

```markdown
## Architecture Notes

### Alchemy AA SDK vs Base Account SDK

- **Base Account** uses custom `@base-org/account` SDK with Coinbase-specific features
- **GateLayer AA Wallet** uses Alchemy's standard Light Account implementation
- Light Account is more lightweight and has lower deployment costs
- Both follow ERC-4337 standard but with different APIs

### Why Light Account?

- **Battle-tested** - Based on Ethereum Foundation's SimpleAccount
- **Gas optimized** - Significantly reduced gas costs vs alternatives
- **Standard compliance** - Full ERC-4337 and ERC-1271 support
- **Audit verified** - Quantstamp audited for production use
- **Multi-chain support** - Works across all EVM chains including GateLayer

### Bundler Configuration

Alchemy AA SDK handles bundling automatically:
- UserOperations are submitted to Alchemy's Bundler
- No manual bundler endpoint configuration needed
- Alchemy GateLayer RPC recommended for best compatibility

## For Edge Cases and Latest API Changes

- **Alchemy AA SDK Docs**: https://docs.alchemy.com/reference/aa-sdk
- **ERC-4337 Specification**: https://eips.ethereum.org/EIPS/eip-4337
- **GateLayer Documentation**: https://www.gatechain.io/docs/GateLayer/Introduction/
- **GateLayer Network Config**: Use @connecting-to-gatelayer-network skill
```

**Step 2: Final SKILL.md validation**

Run:
```bash
# Check file length
wc -l skills/building-with-gatelayer-aa/SKILL.md
```

Expected: ~150-200 lines for main file

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: complete SKILL.md with architecture section"
```

---

## Task 7: Create authentication.md Reference

**Files:**
- Create: `skills/building-with-gatelayer-aa/references/authentication.md`

**Step 1: Write authentication reference**

Create file with:

```markdown
# Authentication (Multi-Provider Sign-In)

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Supported Providers](#supported-providers)
- [Email Authentication](#email-authentication)
- [Social Login](#social-login)
- [Passkeys](#passkeys)
- [Traditional Wallets](#traditional-wallets)
- [Backend Verification](#backend-verification)
- [Framework Integration](#framework-integration)
- [Security Checklist](#security-checklist)

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

## Security Checklist

- [ ] Generate nonces **before** user clicks sign-in button
- [ ] Track used nonces server-side with Set or database
- [ ] Verify signatures on backend (never trust frontend)
- [ ] Use `Cross-Origin-Opener-Policy: same-origin-allow-popups`
- [ ] Set appropriate session/JWT expiry times
- [ ] Include Chain ID in verification to prevent cross-chain replay
- [ ] Validate message format and nonce presence
- [ ] Use HTTPS for all authentication endpoints
```

**Step 2: Validate authentication.md**

Run:
```bash
wc -l skills/building-with-gatelayer-aa/references/authentication.md
```

Expected: ~200-250 lines

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/authentication.md
git commit -m "feat: add authentication reference document"
```

---

## Task 8: Create smart-wallet.md Reference

**Files:**
- Create: `skills/building-with-gatelayer-aa/references/smart-wallet.md`

**Step 1: Write smart wallet reference**

Create file with:

```markdown
# Smart Wallet (Light Account)

## Table of Contents

- [Overview](#overview)
- [Light Account Features](#light-account-features)
- [Creating an Account](#creating-an-account)
- [Importing Existing Account](#importing-existing-account)
- [Account Deployment](#account-deployment)
- [Sending Transactions](#sending-transactions)
- [Balance Checking](#balance-checking)
- [Message Signing](#message-signing)
- [Account Management](#account-management)
- [Security Best Practices](#security-best-practices)

## Overview

Light Account is a production-ready ERC-4337 smart contract account based on Ethereum Foundation's SimpleAccount. It provides gas optimization, security, and compatibility with OpenSea and other protocols.

## Light Account Features

- **Gas Optimized** - Lower deployment and transaction costs
- **ERC-1271 Support** - Signature verification for OpenSea compatibility
- **Ownership Transfer** - Change account owner/key
- **Quantstamp Audited** - Security verified for production use
- **Multi-Chain** - Works on GateLayer and other EVM chains

## Creating an Account

### Basic Setup

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";
import { http } from "viem";

const gatelayerMainnet = {
  id: 10088,
  name: "GateLayer",
  rpcUrls: {
    public: {
      http: ["https://gatelayer-mainnet.gatenode.cc"]
    }
  },
  nativeCurrency: {
    name: "GT",
    symbol: "GT",
    decimals: 18
  }
};

// Create new Light Account
const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
  owner: signer, // From authentication
});

console.log("Account Address:", client.account.address);
// Account is deterministic based on owner
```

### Account Address Determinism

```typescript
// Same owner + same deployment configuration = same address
const owner = signer.address;

// Address is computed before deployment
const predictedAddress = await client.getAddress(owner);
console.log("Predicted Address:", predictedAddress);

// client.account.address will match predictedAddress
```

## Importing Existing Account

```typescript
// Import by account address (requires ownership proof)
const existingClient = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
  accountAddress: "0xExistingLightAccount",
  owner: signer,
});

// Can now use existing account
console.log("Imported Account:", existingClient.account.address);
```

## Account Deployment

### Check Deployment Status

```typescript
// Check if account is deployed onchain
const isDeployed = await client.isAccountDeployed();

if (!isDeployed) {
  console.log("Account not yet deployed");
  console.log("Will be deployed with first transaction");
}
```

### Manual Deployment (Optional)

```typescript
if (!await client.isAccountDeployed()) {
  // Deploy account explicitly
  const deployHash = await client.deployAccount();
  console.log("Deploy Hash:", deployHash);

  // Wait for deployment
  await client.waitForDeployment(deployHash);
}
```

**Note**: Account deploys automatically with first transaction. Manual deployment is optional.

## Sending Transactions

### Simple Transfer

```typescript
const result = await client.sendUserOperation({
  uo: {
    target: "0xRecipientAddress",
    data: "0x", // Empty data for simple transfer
    value: parseEther("0.1"), // 0.1 GT
  },
});

console.log("UserOp Hash:", result.hash);

// Wait for transaction
const receipt = await client.waitForUserOperationTransaction({
  hash: result.hash,
});

console.log("Transaction Hash:", receipt.transactionHash);
```

### Contract Interaction

```typescript
import { encodeFunctionData } from "viem";

const usdcTransferData = encodeFunctionData({
  abi: usdcAbi,
  functionName: "transfer",
  args: ["0xRecipient", parseUnits("100", 6)], // 100 USDC
});

const result = await client.sendUserOperation({
  uo: {
    target: usdcTokenAddress,
    data: usdcTransferData,
    value: 0n,
  },
});
```

## Balance Checking

### Native Balance (GT)

```typescript
const balance = await client.getBalance();
console.log("GT Balance:", formatEther(balance));

// Check balance of any address
const otherBalance = await client.getBalance({
  address: "0xOtherAddress",
});
```

### ERC-20 Balance

```typescript
import { erc20Abi } from "viem";

const balance = await client.readContract({
  address: usdcTokenAddress,
  abi: erc20Abi,
  functionName: "balanceOf",
  args: [client.account.address],
});

console.log("USDC Balance:", formatUnits(balance, 6));
```

## Message Signing

### Sign Message

```typescript
const message = "Sign in to GateLayer App";

const signature = await client.signMessage({
  message,
});

console.log("Signature:", signature);
```

### Sign Typed Data (EIP-712)

```typescript
const domain = {
  name: "GateLayerApp",
  version: "1",
  chainId: 10088,
};

const types = {
  Permit: [
    { name: "owner", type: "address" },
    { name: "spender", type: "address" },
    { name: "value", type: "uint256" },
  ],
};

const value = {
  owner: client.account.address,
  spender: "0xSpender",
  value: parseEther("100"),
};

const signature = await client.signTypedData({
  domain,
  types,
  primaryType: "Permit",
  value,
});
```

## Account Management

### Transfer Ownership

```typescript
// Transfer account to new owner
const newOwner = "0xNewOwnerAddress";

const result = await client.sendUserOperation({
  uo: {
    target: client.account.address,
    data: encodeFunctionData({
      abi: lightAccountAbi,
      functionName: "transferOwnership",
      args: [newOwner],
    }),
    value: 0n,
  },
});
```

### Get Current Owner

```typescript
import { lightAccountAbi } from "./abis";

const owner = await client.readContract({
  address: client.account.address,
  abi: lightAccountAbi,
  functionName: "owner",
});

console.log("Account Owner:", owner);
```

## Security Best Practices

- [ ] **Never expose private keys** - use signer from authentication
- [ ] **Validate addresses** before sending transactions
- [ ] **Check deployment status** before assuming account exists
- [ ] **Use exact amounts** - avoid floating point math with ETH values
- [ ] **Verify transaction data** before signing
- [ ] **Monitor gas prices** to avoid overpaying
- [ ] **Test on testnet first** before mainnet transactions
- [ ] **Backup ownership** - save mnemonic or recovery phrase securely
- [ ] **Use rate limiting** on backend operations
- [ ] **Implement transaction monitoring** for security alerts

## Common Patterns

### Estimate Gas Before Sending

```typescript
const gasEstimate = await client.estimateUserOperationGas({
  uo: {
    target: "0xRecipient",
    data: "0x",
    value: parseEther("0.1"),
  },
});

console.log("Estimated Gas:", gasEstimate);
```

### Wait for Transaction Confirmation

```typescript
const result = await client.sendUserOperation({ /* ... */ });

// Wait with timeout
const receipt = await client.waitForUserOperationTransaction({
  hash: result.hash,
  timeout: 60000, // 60 seconds
});
```

### Handle Transaction Failures

```typescript
try {
  const result = await client.sendUserOperation({ /* ... */ });
} catch (error) {
  if (error.message.includes("insufficient funds")) {
    console.log("Please add GT to your account");
  } else if (error.message.includes("reverted")) {
    console.log("Transaction reverted");
  }
}
```
```

**Step 2: Validate smart-wallet.md**

Run:
```bash
wc -l skills/building-with-gatelayer-aa/references/smart-wallet.md
```

Expected: ~250-300 lines

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/smart-wallet.md
git commit -m "feat: add smart wallet reference document"
```

---

## Task 9: Create paymaster.md Reference

**Files:**
- Create: `skills/building-with-gatelayer-aa/references/paymaster.md`

**Step 1: Write paymaster reference**

Create file with:

```markdown
# Gas Sponsorship (Paymaster)

## Table of Contents

- [Overview](#overview)
- [How Paymasters Work](#how-paymasters-work)
- [Supported Providers](#supported-providers)
- [Alchemy Gas Manager Setup](#alchemy-gas-manager-setup)
- [Sponsorship Policies](#sponsorship-policies)
- [Implementation](#implementation)
- [Cost Monitoring](#cost-monitoring)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)

## Overview

Paymasters enable gasless transactions by sponsoring user gas fees. Users can interact with your application without holding GT, while you cover the costs.

## How Paymasters Work

1. **User initiates transaction** (no GT required)
2. **Paymaster validates** request against policy
3. **Paymaster sponsors gas** for the transaction
4. **Transaction executes** user pays 0 gas
5. **You reimburse** Paymaster for sponsored gas

## Supported Providers

| Provider | ERC-7677 Compatible | Setup Complexity | Features |
|----------|-------------------|------------------|----------|
| Alchemy Gas Manager | ✅ | Low | Dashboard, policies, API |
| Pimlico | ✅ | Medium | Open-source, flexible |
| Gelato Relay | ✅ | Medium | Decentralized network |
| ZeroDev | ✅ | Medium | Open-source, cheap |

## Alchemy Gas Manager Setup

### 1. Create Policy

1. Go to [Alchemy Dashboard](https://dashboard.alchemy.com/)
2. Navigate to Gas Manager
3. Create new policy with rules
4. Copy Policy ID

### 2. Configure Client

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
  owner: signer,
  gasManagerConfig: {
    policyId: process.env.GATELAYER_GAS_POLICY_ID,
  },
});
```

## Sponsorship Policies

### Policy Types

#### 1. Whitelist Mode (Recommended)

Only sponsor transactions from specific addresses:

```typescript
// Alchemy Dashboard Policy Rules
{
  "rules": [
    {
      "type": "whitelist",
      "addresses": [
        "0xVerifiedUser1",
        "0xVerifiedUser2"
      ]
    }
  ]
}
```

#### 2. Limit Mode

Sponsor up to X gas per address/day:

```typescript
{
  "rules": [
    {
      "type": "spendingLimit",
      "limit": "10000000000000000", // 0.01 GT per day
      "period": "daily"
    }
  ]
}
```

#### 3. Transaction Type Filter

Only sponsor specific contract calls:

```typescript
{
  "rules": [
    {
      "type": "contractCall",
      "allowedContracts": [
        "0xMyNFTContract",
        "0xMyTokenContract"
      ],
      "allowedFunctions": [
        "mint",
        "transfer"
      ]
    }
  ]
}
```

#### 4. Global Policy

Sponsor all transactions (use with caution):

```typescript
{
  "rules": [
    {
      "type": "all"
    }
  ]
}
```

**⚠️ Warning**: Global policies can lead to unlimited costs. Use limits.

## Implementation

### Send Sponsored Transaction

```typescript
const result = await client.sendUserOperation({
  uo: {
    target: "0xNFTContract",
    data: encodeFunctionData({
      abi: nftAbi,
      functionName: "mint",
      args: [recipientAddress],
    }),
    value: 0n,
  },
  // Gas sponsorship happens automatically
  // based on gasManagerConfig policy
});

console.log("Sponsored Transaction:", result.hash);
```

### Check Sponsorship Status

```typescript
const gasDetails = await client.estimateUserOperationGas({
  uo: {
    target: "0xContract",
    data: "0x",
    value: 0n,
  },
});

console.log("Gas to be sponsored:", gasDetails);
```

### Conditional Sponsorship

```typescript
// Only sponsor if policy allows
try {
  const result = await client.sendUserOperation({
    uo: { /* transaction */ },
  });
  console.log("Transaction sponsored");
} catch (error) {
  if (error.message.includes("paymaster")) {
    // Fallback: user pays gas
    console.log("Sponsorship unavailable, user pays gas");
    const result = await client.sendUserOperation({
      uo: { /* transaction */ },
      skipGasSponsorship: true,
    });
  }
}
```

## Cost Monitoring

### Alchemy Dashboard

Monitor in real-time:
- Total gas sponsored
- Cost per transaction
- Policy utilization
- Daily/monthly spending

### API Monitoring

```typescript
// Track sponsored transactions in your database
await db.sponsoredTransactions.create({
  data: {
    userAddress,
    transactionHash: result.hash,
    gasSponsored: gasEstimate,
    timestamp: new Date(),
  },
});

// Query for analytics
const totalSponsored = await db.sponsoredTransactions.aggregate({
  _sum: { gasSponsored: true },
  where: {
    timestamp: {
      gte: startOfMonth,
    },
  },
});
```

### Set Budget Alerts

```typescript
// Check budget before sponsoring
const monthlySpend = await getMonthlySpend();

if (monthlySpend >= MONTHLY_BUDGET) {
  throw new Error("Monthly gas budget exceeded");
}
```

## Security Considerations

### Protect Paymaster URL

**❌ NEVER expose in frontend:**

```typescript
// BAD: Paymaster URL visible in browser
const client = createClient({
  paymasterUrl: "https://paymaster.com?secret=KEY"
});
```

**✅ Use backend proxy:**

```typescript
// Frontend requests sponsorship
const response = await fetch("/api/sponsor-gas", {
  method: "POST",
  body: JSON.stringify({ userOperation }),
});

// Backend validates and sponsors
// Backend: server/app.ts
app.post("/api/sponsor-gas", async (req, res) => {
  const { userOperation } = req.body;

  // Validate request
  if (!isValidUserOperation(userOperation)) {
    return res.status(400).json({ error: "Invalid" });
  }

  // Backend sponsors with secret key
  const paymasterData = await alchemy.paymaster.sponsor({
    policyId: process.env.GAS_POLICY_ID,
    userOperation,
  });

  res.json({ paymasterData });
});
```

### Validate Policies

```typescript
// Backend: Enforce additional checks
app.post("/api/sponsor-gas", async (req, res) => {
  const { userAddress, targetContract } = req.body;

  // Check if user is whitelisted
  const isWhitelisted = await db.whitelist.findUnique({
    where: { address: userAddress },
  });

  if (!isWhitelisted) {
    return res.status(403).json({ error: "Not whitelisted" });
  }

  // Check if contract is allowed
  const allowedContracts = [
    "0xNFTContract",
    "0xTokenContract",
  ];

  if (!allowedContracts.includes(targetContract)) {
    return res.status(403).json({ error: "Contract not allowed" });
  }

  // Sponsor transaction
  // ...
});
```

### Prevent Abuse

```typescript
// Rate limiting per user
const sponsorCount = await redis.incr(`sponsor:${userAddress}:${today}`);

if (sponsorCount > MAX_DAILY_SPONSORED_TXS) {
  return res.status(429).json({ error: "Daily limit exceeded" });
}

// Detect unusual patterns
const recentTxCount = await getRecentTransactionCount(userAddress);

if (recentTxCount > SUSPICIOUS_THRESHOLD) {
  // Flag for review
  await flagSuspiciousActivity(userAddress);
  return res.status(403).json({ error: "Flagged for review" });
}
```

## Troubleshooting

### Paymaster Denied Transaction

**Error**: "Paymaster denied request"

**Causes**:
1. Address not whitelisted
2. Spending limit exceeded
3. Contract not allowed
4. Invalid Paymaster configuration

**Solution**:
```typescript
// Check policy status
const policyStatus = await alchemy.gasManager.getPolicyStatus({
  policyId: process.env.GAS_POLICY_ID,
});

console.log("Policy Status:", policyStatus);
```

### Gas Estimation Failed

**Error**: "Gas estimation failed"

**Solution**:
```typescript
// Try without sponsorship
const estimate = await client.estimateUserOperationGas({
  uo: { /* transaction */ },
  skipGasSponsorship: true,
});

// If this works, Paymaster is the issue
```

### Paymaster Timeout

**Error**: "Paymaster request timeout"

**Solution**:
```typescript
// Increase timeout
const client = createLightAccountAlchemyClient({
  // ...
  gasManagerConfig: {
    policyId: process.env.GAS_POLICY_ID,
    timeout: 30000, // 30 seconds
  },
});
```

## Best Practices

1. **Start with whitelist mode** - Only sponsor verified users
2. **Set spending limits** - Prevent unlimited costs
3. **Monitor daily** - Check Alchemy dashboard regularly
4. **Use fallback** - Allow user-paid gas when sponsorship unavailable
5. **Backend validation** - Never trust frontend requests alone
6. **Budget alerts** - Set up alerts for unusual spending
7. **Test on testnet** - Verify policies before mainnet
8. **Document policies** - Keep team informed of rules

## Cost Calculation

### Estimate Monthly Costs

```typescript
// Example: 1000 users, 5 tx/day each, 0.001 GT gas
const dailyTxs = 1000 * 5; // 5000
const monthlyTxs = dailyTxs * 30; // 150,000
const gasPerTx = parseEther("0.001"); // 0.001 GT
const monthlyCost = monthlyTxs * gasPerTx;

console.log("Estimated Monthly Gas Cost:", formatEther(monthlyCost), "GT");
// Output: "Estimated Monthly Gas Cost: 150 GT"
```

### Optimize Costs

- **Batch transactions** - Combine multiple operations
- **Use efficient contracts** - Lower gas functions
- **Set appropriate limits** - Don't over-sponsor
- **Monitor patterns** - Identify waste
- **Consider tiered sponsorship** - More for active users
```

**Step 2: Validate paymaster.md**

Run:
```bash
wc -l skills/building-with-gatelayer-aa/references/paymaster.md
```

Expected: ~300-350 lines

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/paymaster.md
git commit -m "feat: add paymaster reference document"
```

---

## Task 10: Create batch-transactions.md Reference

**Files:**
- Create: `skills/building-with-gatelayer-aa/references/batch-transactions.md`

**Step 1: Write batch transactions reference**

Create file with:

```markdown
# Batch Transactions

## Table of Contents

- [Overview](#overview)
- [How Batching Works](#how-batching-work)
- [Execution Modes](#execution-modes)
- [Building Batches](#building-batches)
- [Atomic Transactions](#atomic-transactions)
- [Status Monitoring](#status-monitoring)
- [Error Handling](#error-handling)
- [Gas Optimization](#gas-optimization)
- [Use Cases](#use-cases)

## Overview

Batch transactions allow executing multiple operations in a single transaction. This improves UX (one confirmation), reduces gas (shared overhead), and enables atomic execution (all succeed or all fail).

## How Batching Works

1. **Define multiple calls** - targets, data, values
2. **Batch into UserOperation** - single operation containing all calls
3. **Submit to Bundler** - Alchemy handles bundling
4. **Execute atomically** - all calls in one transaction
5. **Return combined result** - success or failure for all

## Execution Modes

### 1. Atomic Batch (Recommended)

All calls succeed or all revert together:

```typescript
const batchUo = await client.buildUserOp({
  uo: [
    { target: "0xToken", data: approveCall, value: 0n },
    { target: "0xDex", data: swapCall, value: 0n },
  ],
});

const result = await client.sendUserOperation({
  uo: batchUo,
  atomicExecution: true, // All or nothing
});
```

**Use when**: Operations depend on each other (approve + swap)

### 2. Sequential Execution

Execute in order, stop on first error:

```typescript
const result = await client.sendUserOperation({
  uo: [
    { target: "0xA", data: callA, value: 0n },
    { target: "0xB", data: callB, value: 0n },
    { target: "0xC", data: callC, value: 0n },
  ],
  executionMode: "sequential",
});
```

**Use when**: Order matters but failure is acceptable

### 3. Parallel Execution

Execute independently:

```typescript
const result = await client.sendUserOperation({
  uo: [
    { target: "0xA", data: callA, value: 0n },
    { target: "0xB", data: callB, value: 0n },
  ],
  executionMode: "parallel",
});
```

**Use when**: Operations are independent

## Building Batches

### Simple Batch

```typescript
import { encodeFunctionData, parseEther } from "viem";

const batchUo = await client.buildUserOp({
  uo: [
    {
      target: "0xRecipient1",
      data: "0x",
      value: parseEther("0.1"),
    },
    {
      target: "0xRecipient2",
      data: "0x",
      value: parseEther("0.2"),
    },
    {
      target: "0xRecipient3",
      data: "0x",
      value: parseEther("0.15"),
    },
  ],
});

const result = await client.sendUserOperation({ uo: batchUo });
```

### Contract Interactions

```typescript
import { usdcAbi, nftAbi } from "./abis";

const batchUo = await client.buildUserOp({
  uo: [
    // 1. Approve USDC
    {
      target: usdcAddress,
      data: encodeFunctionData({
        abi: usdcAbi,
        functionName: "approve",
        args: [spenderAddress, maxUint256],
      }),
      value: 0n,
    },
    // 2. Mint NFT (uses USDC approval)
    {
      target: nftAddress,
      data: encodeFunctionData({
        abi: nftAbi,
        functionName: "mint",
        args: [recipientAddress, tokenId],
      }),
      value: 0n,
    },
  ],
});

const result = await client.sendUserOperation({
  uo: batchUo,
  atomicExecution: true, // Must succeed together
});
```

### Complex Multi-Step DeFi

```typescript
// Uniswap-style: approve + swap + stake
const batchUo = await client.buildUserOp({
  uo: [
    // 1. Approve token for router
    {
      target: tokenInAddress,
      data: encodeFunctionData({
        abi: erc20Abi,
        functionName: "approve",
        args: [routerAddress, amountIn],
      }),
      value: 0n,
    },
    // 2. Swap tokens
    {
      target: routerAddress,
      data: encodeFunctionData({
        abi: routerAbi,
        functionName: "exactInputSingle",
        args: [{
          tokenIn: tokenInAddress,
          tokenOut: tokenOutAddress,
          amountIn,
          amountOutMinimum,
          recipient: client.account.address,
          deadline: Math.floor(Date.now() / 1000) + 1800,
        }],
      }),
      value: 0n,
    },
    // 3. Stake output tokens
    {
      target: stakingContractAddress,
      data: encodeFunctionData({
        abi: stakingAbi,
        functionName: "stake",
        args: [amountOutMinimum],
      }),
      value: 0n,
    },
  ],
});

const result = await client.sendUserOperation({
  uo: batchUo,
  atomicExecution: true,
});
```

## Atomic Transactions

### Why Atomic?

**Without atomic batching**:
1. Approve succeeds ✅
2. Swap fails ❌
3. Approval wasted, user must revoke

**With atomic batching**:
1. Approve + swap in one transaction
2. If swap fails, approval also reverts
3. Clean state, no cleanup needed

### Force Atomic Execution

```typescript
const result = await client.sendUserOperation({
  uo: batchUo,
  atomicExecution: true, // Required for DeFi operations
});

// If any call fails, entire batch reverts
```

### Check Atomic Support

```typescript
// Verify Light Account supports atomic batching
const supportsAtomic = await client.getCapabilities();
console.log("Atomic Batch Support:", supportsAtomic.atomic);
```

## Status Monitoring

### Monitor Execution

```typescript
const result = await client.sendUserOperation({ uo: batchUo });

// Poll for completion
const status = await client.getUserOperationStatus({
  hash: result.hash,
});

// Status values: "pending", "confirmed", "reverted"
while (status.status === "pending") {
  await new Promise(r => setTimeout(r, 2000));
  status = await client.getUserOperationStatus({
    hash: result.hash,
  });
}

if (status.status === "confirmed") {
  console.log("All batch operations succeeded");
} else if (status.status === "reverted") {
  console.log("Batch reverted - atomicity maintained");
}
```

### Get Individual Call Results

```typescript
const receipt = await client.waitForUserOperationTransaction({
  hash: result.hash,
});

// Each call result available
receipt.receipts.forEach((callReceipt, index) => {
  console.log(`Call ${index}:`, callReceipt.status);
  if (callReceipt.status === "success") {
    console.log(`Gas used: ${callReceipt.gasUsed}`);
  } else {
    console.log(`Revert reason: ${callReceipt.revertReason}`);
  }
});
```

## Error Handling

### Partial Failure (Non-Atomic)

```typescript
try {
  const result = await client.sendUserOperation({
    uo: batchUo,
    executionMode: "sequential", // Not atomic
  });
} catch (error) {
  // Find which call failed
  if (error.failedCallIndex !== undefined) {
    console.log(`Call ${error.failedCallIndex} failed`);
    console.log("Reason:", error.revertReason);

    // Calls before succeeded, after never executed
  }
}
```

### Complete Failure (Atomic)

```typescript
try {
  const result = await client.sendUserOperation({
    uo: batchUo,
    atomicExecution: true,
  });
} catch (error) {
  // Entire batch failed
  console.log("Batch reverted:", error.message);

  // No partial state changes
  console.log("Atomicity maintained - clean revert");
}
```

### Handle Specific Errors

```typescript
try {
  await client.sendUserOperation({ uo: batchUo });
} catch (error) {
  if (error.message.includes("insufficient funds")) {
    console.log("Batch needs more GT for gas");
  } else if (error.message.includes("revert")) {
    console.log("One or more calls reverted");
    // Debug: execute calls individually to find culprit
  } else if (error.message.includes("batch too large")) {
    console.log("Too many calls - split into smaller batches");
  }
}
```

## Gas Optimization

### Gas Savings from Batching

**Separate transactions**:
- Call 1: 50,000 gas
- Call 2: 45,000 gas
- Call 3: 52,000 gas
- **Total: 147,000 gas**

**Batched transaction**:
- Fixed overhead: 21,000 gas
- Call 1: 50,000 gas
- Call 2: 45,000 gas
- Call 3: 52,000 gas
- **Total: 168,000 gas** ✅ 21,000 gas saved!

### Optimal Batch Size

```typescript
// Test different batch sizes
const batchSize = 10;
const calls = Array.from({ length: batchSize }, (_, i) => ({
  target: `0xContract${i}`,
  data: "0x",
  value: 0n,
}));

const gasEstimate = await client.estimateUserOperationGas({
  uo: calls,
});

const gasPerCall = Number(gasEstimate) / batchSize;
console.log(`Avg gas per call in batch: ${gasPerCall}`);
```

### Batch Sizing Strategy

```typescript
// If batch too large, split it
async function executeInBatches(calls, maxBatchSize = 20) {
  const batches = [];
  for (let i = 0; i < calls.length; i += maxBatchSize) {
    batches.push(calls.slice(i, i + maxBatchSize));
  }

  for (const batch of batches) {
    const result = await client.sendUserOperation({
      uo: batch,
      atomicExecution: true,
    });
    console.log(`Batch ${batches.indexOf(batch)}:`, result.hash);
  }
}
```

## Use Cases

### 1. DeFi Trading

```typescript
// Approve + swap in one transaction
const tradeBatch = await client.buildUserOp({
  uo: [
    {
      target: tokenAddress,
      data: encodeFunctionData({
        abi: erc20Abi,
        functionName: "approve",
        args: [dexAddress, amount],
      }),
      value: 0n,
    },
    {
      target: dexAddress,
      data: encodeFunctionData({
        abi: dexAbi,
        functionName: "swap",
        args: [tokenIn, tokenOut, amount],
      }),
      value: 0n,
    },
  ],
});
```

### 2. NFT Minting with Payment

```typescript
// Mint + pay in one transaction
const mintBatch = await client.buildUserOp({
  uo: [
    {
      target: paymentTokenAddress,
      data: encodeFunctionData({
        abi: erc20Abi,
        functionName: "approve",
        args: [nftAddress, mintPrice],
      }),
      value: 0n,
    },
    {
      target: nftAddress,
      data: encodeFunctionData({
        abi: nftAbi,
        functionName: "mint",
        args: [recipient, tokenId],
      }),
      value: 0n,
    },
  ],
});
```

### 3. Multi-Transfer

```typescript
// Send to multiple addresses
const recipients = [
  { address: "0xA", amount: parseEther("0.1") },
  { address: "0xB", amount: parseEther("0.2") },
  { address: "0xC", amount: parseEther("0.15") },
];

const transferBatch = await client.buildUserOp({
  uo: recipients.map(r => ({
    target: r.address,
    data: "0x",
    value: r.amount,
  })),
});
```

### 4. Contract Deployment + Setup

```typescript
// Deploy contract + initialize
const deployBatch = await client.buildUserOp({
  uo: [
    {
      target: factoryAddress,
      data: encodeFunctionData({
        abi: factoryAbi,
        functionName: "deploy",
        args: [constructorArgs],
      }),
      value: 0n,
    },
    {
      target: deployedContractAddress,
      data: encodeFunctionData({
        abi: contractAbi,
        functionName: "initialize",
        args: [initArgs],
      }),
      value: 0n,
    },
  ],
});
```

### 5. Airdrop

```typescript
// Airdrop tokens to 100 users
const airdropRecipients = [...]; // 100 addresses

const airdropBatches = [];
const batchSize = 20;

for (let i = 0; i < airdropRecipients.length; i += batchSize) {
  const batch = airdropRecipients.slice(i, i + batchSize).map(recipient => ({
    target: tokenAddress,
    data: encodeFunctionData({
      abi: erc20Abi,
      functionName: "transfer",
      args: [recipient.address, recipient.amount],
    }),
    value: 0n,
  }));

  airdropBatches.push(batch);
}

// Execute all batches
for (const batch of airdropBatches) {
  await client.sendUserOperation({ uo: batch });
}
```

## Best Practices

1. **Use atomic batching** for dependent operations
2. **Test each call individually** before batching
3. **Estimate gas** for entire batch
4. **Monitor status** after submission
5. **Handle failures** gracefully with fallbacks
6. **Optimize batch size** for gas efficiency
7. **Use Paymaster** with batches for gasless UX
8. **Log detailed results** for debugging
9. **Consider rate limits** on target contracts
10. **Validate all inputs** before batching

## Limitations

- **Max batch size**: Limited by block gas limit (~50-100 calls)
- **Revert data**: Limited space for failure reasons
- **Ordering**: Atomic batches must succeed together
- **Gas estimation**: More complex for batches
- **Debugging**: Harder to isolate failing calls

## Alternatives

### Multicall3 Contract

```typescript
// If Light Account batching unavailable
import { multicall3Abi } from "./abis";

const calls = [
  { target: "0xA", callData: "0x..." },
  { target: "0xB", callData: "0x..." },
];

await client.writeContract({
  address: multicall3Address,
  abi: multicall3Abi,
  functionName: "aggregate3",
  args: [calls],
});
```

**Trade-offs**: Less gas efficient, no atomicity, external dependency
```

**Step 2: Validate batch-transactions.md**

Run:
```bash
wc -l skills/building-with-gatelayer-aa/references/batch-transactions.md
```

Expected: ~350-400 lines

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/batch-transactions.md
git commit -m "feat: add batch transactions reference document"
```

---

## Task 11: Create troubleshooting.md Reference

**Files:**
- Create: `skills/building-with-gatelayer-aa/references/troubleshooting.md`

**Step 1: Write troubleshooting reference**

Create file with:

```markdown
# Troubleshooting

## Table of Contents

- [Common Errors](#common-errors)
- [Popup Issues](#popup-issues)
- [Gas Problems](#gas-problems)
- [Paymaster Failures](#paymaster-failures)
- [Batch Execution Issues](#batch-execution-issues)
- [Account Problems](#account-problems)
- [Network Issues](#network-issues)
- [Debugging Tools](#debugging-tools)
- [FAQ](#faq)

## Common Errors

### Error: "Popup blocked by browser"

**Cause**: Nonce generated after user interaction

**Solution**:
```typescript
// ❌ WRONG: Nonce generated on click
<button onClick={async () => {
  const nonce = crypto.randomUUID(); // Too late!
  await signIn({ nonce });
}}>

// ✅ CORRECT: Nonce generated before click
const nonce = crypto.randomUUID(); // On page load
<button onClick={() => signIn({ nonce })}>
```

**Additional Fix**:
```html
<!-- Add to HTML head -->
<meta http-equiv="Cross-Origin-Opener-Policy" content="same-origin-allow-popups">
```

---

### Error: "Insufficient funds"

**Cause**: Balance check doesn't consider auxiliary funds

**Solution**:
```typescript
// ❌ WRONG: Block based on onchain balance
const balance = await client.getBalance();
if (balance < requiredAmount) {
  return showInsufficientFundsError();
}

// ✅ CORRECT: Let wallet handle funding
// Skip balance check, wallet may have off-chain funds
try {
  await client.sendUserOperation({ uo: transaction });
} catch (error) {
  if (error.message.includes("insufficient funds")) {
    showInsufficientFundsError(); // Now show error
  }
}
```

---

### Error: "Paymaster request denied"

**Cause**: Policy configuration issue

**Debug Steps**:
```typescript
// 1. Check policy ID is correct
console.log("Policy ID:", process.env.GAS_POLICY_ID);

// 2. Verify policy exists
const policy = await alchemy.gasManager.getPolicy({
  policyId: process.env.GAS_POLICY_ID,
});
console.log("Policy:", policy);

// 3. Check address is whitelisted (if using whitelist)
const isWhitelisted = policy.whitelist.includes(userAddress);
console.log("Whitelisted:", isWhitelisted);

// 4. Check spending limits
const usage = await alchemy.gasManager.getUsage({
  policyId: process.env.GAS_POLICY_ID,
});
console.log("Gas Used:", usage);
```

---

### Error: "Invalid chain ID"

**Cause**: Chain ID configuration mismatch

**Solution**:
```typescript
// ❌ WRONG: Hardcoded chain ID
const chain = { id: 1 }; // Ethereum!

// ✅ CORRECT: Use connecting-to-gatelayer-network skill
import { gatelayerMainnet } from "@gatechain/gatelayer-skills";

const chain = gatelayerMainnet; // { id: 10088, ... }

// Verify chain ID
console.log("Expected Chain ID:", 10088);
console.log("Current Chain ID:", await client.getChainId());
```

---

### Error: "Account not deployed"

**Cause**: First transaction, account needs deployment

**Solution**:
```typescript
// Option 1: Let first transaction deploy account
const result = await client.sendUserOperation({
  uo: { target: "0xContract", data: "0x", value: 0n },
});
// Account deploys automatically

// Option 2: Deploy explicitly
if (!await client.isAccountDeployed()) {
  console.log("Deploying account...");
  await client.deployAccount();
}

// Option 3: Wait for deployment
const deployHash = await client.deployAccount();
await client.waitForDeployment(deployHash);
console.log("Account deployed!");
```

---

### Error: "Signature verification failed"

**Cause**: ERC-1271 signature format issue

**Solution**:
```typescript
// ❌ WRONG: Manual verification
const isValid = recoverAddress(message, signature) === address;

// ✅ CORRECT: Use viem (handles ERC-6492)
import { verifyMessage } from "viem";

const isValid = await verifyMessage({
  address,
  message,
  signature,
});
```

---

### Error: "Transaction reverted"

**Cause**: Contract call failed

**Debug Steps**:
```typescript
try {
  await client.sendUserOperation({ uo: transaction });
} catch (error) {
  // Get detailed revert reason
  const reason = error.reason || error.data;

  if (reason.includes("InsufficientBalance")) {
    console.log("Token balance too low");
  } else if (reason.includes("NotAllowed")) {
    console.log("Contract not allowed to call");
  } else {
    console.log("Revert reason:", reason);
  }

  // Try simulating to get more details
  try {
    const simulation = await client.simulateUserOperation({
      uo: transaction,
    });
    console.log("Simulation result:", simulation);
  } catch (simError) {
    console.log("Simulation error:", simError.message);
  }
}
```

---

## Popup Issues

### Popup doesn't appear

**Check 1**: Nonce timing
```typescript
// Generate nonce BEFORE render
useEffect(() => {
  setNonce(crypto.randomUUID());
}, []);

// Use pre-generated nonce
<button onClick={() => signIn({ nonce })}>
```

**Check 2**: Cross-Origin-Opener-Policy
```typescript
// Check headers
fetch(window.location.href).then(r => {
  console.log("COOP:", r.headers.get("Cross-Origin-Opener-Policy"));
  // Should be "same-origin-allow-popups"
});
```

**Check 3**: Browser console for errors
```typescript
// Add error handling
try {
  await signIn();
} catch (error) {
  if (error.message.includes("popup")) {
    console.log("Popup blocked - check browser settings");
  }
}
```

---

## Gas Problems

### Gas estimation too high

**Solution**:
```typescript
// 1. Check individual call gas
for (const call of batchCalls) {
  const gas = await client.estimateContractGas({
    address: call.target,
    abi: callAbi,
    functionName: call.functionName,
    args: call.args,
  });
  console.log(`Gas for ${call.functionName}:`, gas);
}

// 2. Try without Paymaster
const gasWithout = await client.estimateUserOperationGas({
  uo: call,
  skipGasSponsorship: true,
});

const gasWith = await client.estimateUserOperationGas({
  uo: call,
});

console.log("Without Paymaster:", gasWithout);
console.log("With Paymaster:", gasWith);
```

### Out of gas errors

**Solution**:
```typescript
// Increase gas limit buffer
const gasEstimate = await client.estimateUserOperationGas({
  uo: call,
});

const result = await client.sendUserOperation({
  uo: call,
  gasLimit: gasEstimate * 120n / 100n, // 20% buffer
});
```

---

## Paymaster Failures

### Paymaster timeout

**Solution**:
```typescript
const client = createLightAccountAlchemyClient({
  // ...
  gasManagerConfig: {
    policyId: process.env.GAS_POLICY_ID,
    timeout: 60000, // Increase timeout to 60s
  },
});
```

### Paymaster returns invalid data

**Debug**:
```typescript
// Check Paymaster response
const paymasterData = await alchemy.paymaster.getPaymasterData({
  policyId: process.env.GAS_POLICY_ID,
  userOperation,
});

console.log("Paymaster Data:", paymasterData);

// Check if data is valid
if (!paymasterData.paymasterAndData) {
  console.log("Paymaster denied sponsorship");
}
```

---

## Batch Execution Issues

### Batch too large

**Error**: "Batch exceeds gas limit"

**Solution**:
```typescript
// Split into smaller batches
function splitBatch(calls, maxSize = 20) {
  const batches = [];
  for (let i = 0; i < calls.length; i += maxSize) {
    batches.push(calls.slice(i, i + maxSize));
  }
  return batches;
}

const batches = splitBatch(largeBatch, 20);
for (const batch of batches) {
  await client.sendUserOperation({ uo: batch });
}
```

### One call fails in batch

**Debug**:
```typescript
// Execute calls individually to find culprit
for (const call of batchCalls) {
  try {
    await client.sendUserOperation({
      uo: [call],
      atomicExecution: false,
    });
    console.log("Call succeeded:", call.target);
  } catch (error) {
    console.log("Call failed:", call.target);
    console.log("Error:", error.message);
  }
}
```

---

## Account Problems

### Can't import existing account

**Debug**:
```typescript
// 1. Verify address is Light Account
const bytecode = await client.getBytecode({
  address: accountAddress,
});

if (!bytecode || bytecode === "0x") {
  console.log("Account not deployed");
}

// 2. Check if it's Light Account bytecode
const isLightAccount = bytecode.includes(
  "0x..." // Light Account init code
);
console.log("Is Light Account:", isLightAccount);

// 3. Verify ownership
try {
  const signature = await client.signMessage({ message: "test" });
  const isValid = await verifyMessage({
    address: accountAddress,
    message: "test",
    signature,
  });
  console.log("Ownership verified:", isValid);
} catch (error) {
  console.log("Ownership verification failed - wrong signer");
}
```

### Account address mismatch

**Cause**: Different deployment parameters

**Solution**:
```typescript
// Ensure consistent parameters
const client1 = createLightAccountAlchemyClient({
  apiKey: "key1",
  chain,
  owner: signer,
});

const client2 = createLightAccountAlchemyClient({
  apiKey: "key2", // Different API key!
  chain,
  owner: signer,
});

console.log("Address 1:", client1.account.address);
console.log("Address 2:", client2.account.address);
// These will be DIFFERENT!

// Use same API key for same account
```

---

## Network Issues

### RPC timeout

**Solution**:
```typescript
const client = createLightAccountAlchemyClient({
  // ...
  transport: http("https://gatelayer-mainnet.gatenode.cc", {
    timeout: 60_000, // 60 second timeout
    retryCount: 3,
  }),
});
```

### Wrong network selected

**Check**:
```typescript
const currentChainId = await client.getChainId();
console.log("Current Chain ID:", currentChainId);
console.log("Expected Chain ID:", 10088); // GateLayer Mainnet

if (currentChainId !== 10088) {
  console.log("Please switch to GateLayer Mainnet");
  // Prompt user to switch network
}
```

---

## Debugging Tools

### Enable debug logging

```typescript
import { debug } from "@alchemy/aa-core";

debug.enabled = true; // Enable detailed logs
```

### Simulate before sending

```typescript
const simulation = await client.simulateUserOperation({
  uo: transaction,
});

console.log("Simulation result:", simulation);
// Check for revert reasons before spending gas
```

### Estimate gas

```typescript
const gasEstimate = await client.estimateUserOperationGas({
  uo: transaction,
});

console.log("Estimated gas:", gasEstimate);
// Check if gas is reasonable before sending
```

### Check account deployment

```typescript
const isDeployed = await client.isAccountDeployed();
console.log("Account deployed:", isDeployed);

if (!isDeployed) {
  const deployCost = await client.estimateDeploymentGas();
  console.log("Deployment cost:", deployCost);
}
```

### Get account details

```typescript
const account = await client.getAccount();
console.log("Account:", {
  address: account.address,
  owner: account.owner,
  deploymentTx: account.deploymentTx,
});
```

---

## FAQ

### Q: Do I need GT for first transaction?

**A**: If using Paymaster, no GT needed. If not, need GT for gas + account deployment (~0.001-0.002 GT).

### Q: Can I use other Paymasters?

**A**: Yes, Alchemy AA SDK supports any ERC-7677 compliant Paymaster (Pimlico, Gelato, ZeroDev).

### Q: How long does account deployment take?

**A**: Typically 2-5 seconds on GateLayer. Happens automatically with first transaction.

### Q: Can I recover lost account?

**A**: If you have the owner signer (private key, mnemonic, OAuth), you can recreate account. Lost signer = lost account.

### Q: Is Light Account upgradeable?

**A**: No, Light Account is not upgradeable. For upgradeable accounts, use Modular Account (future support).

### Q: What's the max batch size?

**A**: Limited by block gas limit. Typically 20-50 calls depending on complexity.

### Q: How do I handle network switching?

**A**: Reinitialize client with new chain config:
```typescript
const client = createLightAccountAlchemyClient({
  chain: isMainnet ? gatelayerMainnet : gatelayerTestnet,
  // ...
});
```

### Q: Can I use without Alchemy API key?

**A**: No, Alchemy AA SDK requires API key for Bundler and optional Paymaster services.

### Q: How much does Paymaster cost?

**A**: You pay sponsored gas fees. Typical cost: 0.001-0.01 GT per transaction depending on complexity.

### Q: Is this production-ready?

**A**: Yes, Light Account is Quantstamp audited. Test thoroughly on GateLayer Testnet first.

### Q: Where can I get help?

**A**:
- Alchemy Docs: https://docs.alchemy.com/reference/aa-sdk
- GateLayer Docs: https://www.gatechain.io/docs/GateLayer/Introduction/
- GateLayer Discord: Community support
- GitHub Issues: Bug reports

---

## Getting Additional Help

If issues persist:

1. **Enable debug logging** and share logs
2. **Check Alchemy status page**: https://status.alchemy.com/
3. **Verify GateLayer network status**
4. **Search existing GitHub issues**
5. **Create minimal reproducible example**
6. **Share error messages and stack traces**

## Quick Diagnostic Script

```typescript
async function diagnose() {
  console.log("=== GateLayer AA Wallet Diagnostic ===");

  // 1. Check network
  const chainId = await client.getChainId();
  console.log("Chain ID:", chainId, "(Expected: 10088)");

  // 2. Check account
  const address = client.account.address;
  console.log("Account:", address);

  const isDeployed = await client.isAccountDeployed();
  console.log("Deployed:", isDeployed);

  // 3. Check balance
  const balance = await client.getBalance();
  console.log("Balance:", formatEther(balance), "GT");

  // 4. Check API key
  console.log("API Key configured:", !!process.env.ALCHEMY_API_KEY);

  // 5. Check Paymaster
  console.log("Paymaster configured:", !!process.env.GATELAYER_GAS_POLICY_ID);

  // 6. Test simple call
  try {
    const gasEstimate = await client.estimateUserOperationGas({
      uo: {
        target: address,
        data: "0x",
        value: 0n,
      },
    });
    console.log("Gas estimate OK:", gasEstimate);
  } catch (error) {
    console.log("Gas estimate FAILED:", error.message);
  }

  console.log("=== Diagnostic Complete ===");
}

diagnose();
```
```

**Step 2: Validate troubleshooting.md**

Run:
```bash
wc -l skills/building-with-gatelayer-aa/references/troubleshooting.md
```

Expected: ~400-450 lines

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/troubleshooting.md
git commit -m "feat: add troubleshooting reference document"
```

---

## Task 12: Update Main README.md

**Files:**
- Modify: `README.md`

**Step 1: Add new skill to README table**

Add row to skills table in README.md:

```markdown
| [Building with GateLayer AA Wallet](./skills/building-with-gatelayer-aa/SKILL.md) | Integrates Alchemy AA SDK for GateLayer smart wallet functionality including authentication, gas sponsorship, and batch transactions. |
```

**Step 2: Verify README syntax**

Run:
```bash
grep -A 5 "Available Skills" README.md | head -20
```

Expected: Updated skills table visible

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add GateLayer AA Wallet skill to README"
```

---

## Task 13: Final Validation and Testing

**Files:**
- Test: All skill files

**Step 1: Validate all markdown files**

Run:
```bash
# Check all markdown files exist
ls -la skills/building-with-gatelayer-aa/
ls -la skills/building-with-gatelayer-aa/references/

# Count lines in each file
wc -l skills/building-with-gatelayer-aa/SKILL.md
wc -l skills/building-with-gatelayer-aa/references/*.md
```

Expected: All files present with appropriate line counts

**Step 2: Check for broken links**

Run:
```bash
# Check internal links
grep -r "\](references/" skills/building-with-gatelayer-aa/SKILL.md

# Should show all reference links
```

Expected: All reference files linked correctly

**Step 3: Validate YAML front matter**

Run:
```bash
# Check front matter in main file
head -10 skills/building-with-gatelayer-aa/SKILL.md
```

Expected: Valid YAML front matter with name and description

**Step 4: Final commit**

```bash
git add skills/building-with-gatelayer-aa/ README.md
git commit -m "feat: complete GateLayer AA Wallet skill implementation

- Add main SKILL.md with quick start and feature reference
- Add authentication.md for multi-provider sign-in
- Add smart-wallet.md for Light Account management
- Add paymaster.md for gas sponsorship
- Add batch-transactions.md for batched operations
- Add troubleshooting.md for debugging and FAQ
- Update README.md with new skill

Based on Base Account skill structure, adapted for GateLayer
with Alchemy AA SDK and GateLayer network configuration."
```

---

## Implementation Complete

All tasks complete! The GateLayer AA Wallet skill is now fully implemented with:

✅ Main SKILL.md with quick start guide
✅ Authentication reference (Email, Social, Passkey, Wallets)
✅ Smart Wallet reference (Light Account creation and management)
✅ Paymaster reference (Gas sponsorship setup and policies)
✅ Batch Transactions reference (Atomic batching and optimization)
✅ Troubleshooting reference (Common errors and debugging)
✅ Updated README.md

**Next Steps**:
1. Test skill installation: `npx skills add ./skills/building-with-gatelayer-aa`
2. Verify AI agent can invoke skill correctly
3. Test example code on GateLayer Testnet
4. Gather feedback and iterate
