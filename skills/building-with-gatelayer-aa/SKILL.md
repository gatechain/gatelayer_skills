---
name: building-with-gatelayer-aa
description: Integrates Alchemy AA SDK for GateLayer smart wallet functionality. Covers Light Account creation, multi-provider authentication (Email, Social, Passkey), and batch transactions. Use when building apps with smart wallet authentication, sponsored transactions, or batched operations on GateLayer. Covers phrases like "create AA wallet on GateLayer", "GateLayer smart wallet", "Gasless transactions GateLayer", "Paymaster setup", "batch transactions GateLayer", or "account abstraction GateLayer".
---

# Building with GateLayer AA Wallet

GateLayer AA Wallet provides ERC-4337 smart wallet functionality using Alchemy's Light Account. Enable passwordless authentication, gasless transactions, and batched operations on GateLayer Mainnet and Testnet.

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

## Feature References

Read the reference for the feature you're implementing:

| Feature | Reference | When to Read |
|---------|-----------|-------------|
| Authentication | [references/authentication.md](references/authentication.md) | Email/Social/Passkey login, backend verification, multi-provider setup |
| Smart Wallet | [references/smart-wallet.md](references/smart-wallet.md) | Light Account creation, import, management, signing |
| Batch Transactions | [references/batch-transactions.md](references/batch-transactions.md) | Atomic batching, multi-call execution, gas optimization |
| Troubleshooting | [references/troubleshooting.md](references/troubleshooting.md) | Debugging, common errors, FAQ, best practices |

## Critical Requirements

### Security

- **Track transaction IDs** to prevent replay attacks - use database with unique constraints
- **Verify sender matches authenticated user** to prevent impersonation
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
