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
