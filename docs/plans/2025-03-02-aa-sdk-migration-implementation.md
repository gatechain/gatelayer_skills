# AA SDK Migration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Migrate building-with-gatelayer-aa skill from Alchemy AA SDK to @aa-sdk/core using GateLayer's official RPC and Bundler infrastructure.

**Architecture:** Replace Alchemy-specific SDK with open-source @aa-sdk/core, removing dependency on Alchemy API Keys and using GateLayer's native ERC-4337 infrastructure.

**Tech Stack:** @aa-sdk/core, viem, TypeScript, GateLayer Testnet

---

## Phase 1: Setup and Infrastructure

### Task 1: Create local test project directory

**Files:**
- Create: `../aa-skill-test/` (sibling to gatelayer_skills, NOT in git)
- Create: `../aa-skill-test/.gitignore`

**Step 1: Create .gitignore**

```bash
mkdir -p ../aa-skill-test
```

**Step 2: Write .gitignore**

```text
node_modules
.env
*.log
.DS_Store
dist
```

Run: `cat ../aa-skill-test/.gitignore`
Expected: File contains above content

**Step 3: No commit needed**

This is a local test project only, not committed to git.

---

### Task 2: Initialize test project package.json

**Files:**
- Create: `../aa-skill-test/package.json`

**Step 1: Create package.json**

```json
{
  "name": "aa-skill-test",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "test:01": "bun run src/tests/01-create-account.ts",
    "test:02": "bun run src/tests/02-send-tx.ts",
    "test:03": "bun run src/tests/03-batch.ts",
    "test:04": "bun run src/tests/04-paymaster.ts"
  },
  "dependencies": {
    "@aa-sdk/core": "^1.2.0",
    "viem": "^2.21.54"
  },
  "devDependencies": {
    "@types/node": "^22.10.5",
    "typescript": "^5.7.2"
  }
}
```

**Step 2: Install dependencies**

Run: `cd ../aa-skill-test && bun install`

Expected: Dependencies installed successfully

**Step 3: No commit needed**

---

### Task 3: Create TypeScript configuration

**Files:**
- Create: `../aa-skill-test/tsconfig.json`

**Step 1: Create tsconfig.json**

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ES2022",
    "moduleResolution": "bundler",
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "outDir": "./dist",
    "rootDir": "./src"
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

**Step 4: No commit needed**

---

### Task 4: Create environment template

**Files:**
- Create: `../aa-skill-test/.env.example`
- Create: `../aa-skill-test/.env`

**Step 1: Create .env.example**

```text
# GateLayer Testnet Configuration
GATELAYER_TESTNET_RPC=https://gatelayer-testnet.gatenode.cc
GATELAYER_TESTNET_CHAIN_ID=10087

# Wallet Configuration
PRIVATE_KEY=your_private_key_here

# Test Configuration
TEST_RECIPIENT=0x742d35Cc6634C0532925a3b844Bc9e7595f0bEb
```

**Step 2: Create actual .env with user's values**

Run: `cp .env.example .env`
Then edit .env with actual values

**Step 3: No commit needed**

---

### Task 5: Create GateLayer chain configuration

**Files:**
- Create: `../aa-skill-test/src/config/chains.ts`

**Step 1: Create chains.ts**

```typescript
import { defineChain } from "viem";

/**
 * GateLayer Mainnet Chain Configuration
 *
 * Chain ID: 10088
 * RPC: https://gatelayer-mainnet.gatenode.cc
 * Bundler: Same as RPC (ERC-4337 methods)
 * Explorer: https://www.gatescan.org/gatelayer
 * Currency: GT
 */
export const gatelayerMainnet = defineChain({
  id: 10088,
  name: "GateLayer",
  nativeCurrency: {
    decimals: 18,
    name: "GT",
    symbol: "GT",
  },
  rpcUrls: {
    default: {
      http: ["https://gatelayer-mainnet.gatenode.cc"],
    },
    public: {
      http: ["https://gatelayer-mainnet.gatenode.cc"],
    },
  },
  blockExplorers: {
    default: {
      name: "GateScan",
      url: "https://www.gatescan.org/gatelayer",
    },
  },
  contracts: {
    // ERC-4337 Entry Point v0.6.0
    EntryPoint: {
      0.6: {
        address: "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789" as const,
      },
    },
  },
});

/**
 * GateLayer Testnet Chain Configuration
 *
 * Chain ID: 10087
 * RPC: https://gatelayer-testnet.gatenode.cc
 * Bundler: Same as RPC (ERC-4337 methods)
 * Explorer: https://www.gatescan.org/gatelayer-testnet
 * Currency: GT
 */
export const gatelayerTestnet = defineChain({
  id: 10087,
  name: "GateLayer Testnet",
  nativeCurrency: {
    decimals: 18,
    name: "GT",
    symbol: "GT",
  },
  rpcUrls: {
    default: {
      http: ["https://gatelayer-testnet.gatenode.cc"],
    },
    public: {
      http: ["https://gatelayer-testnet.gatenode.cc"],
    },
  },
  blockExplorers: {
    default: {
      name: "GateScan Testnet",
      url: "https://www.gatescan.org/gatelayer-testnet",
    },
  },
  contracts: {
    // ERC-4337 Entry Point v0.6.0
    EntryPoint: {
      0.6: {
        address: "0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789" as const,
      },
    },
  },
  testnet: true,
});
```

**Step 2: No commit needed**

---

## Phase 2: Update Skill Files

### Task 6: Update SKILL.md header and dependencies

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md:1-145`

**Step 1: Update frontmatter and Quick Start**

Replace the header and Quick Start section:

```markdown
---
name: building-with-gatelayer-aa
description: Use Alchemy's aa-sdk with GateLayer network using official RPC and Bundler. Provides ready-to-use configuration for creating smart account clients, bundler clients, and modular accounts on GateLayer Mainnet and Testnet. Covers phrases like "create AA wallet on GateLayer", "GateLayer smart wallet", "Gasless transactions GateLayer", "Paymaster setup", "batch transactions GateLayer", or "account abstraction GateLayer".
---

# Building with GateLayer AA Wallet

GateLayer AA Wallet provides ERC-4337 smart wallet functionality using Alchemy's open-source `@aa-sdk/core`. Enable passwordless authentication, gasless transactions, and batched operations on GateLayer Mainnet and Testnet using GateLayer's official infrastructure.

## Network Configuration

GateLayer supports ERC-4337 Account Abstraction. Use the network configs below:

| Network | Chain ID | RPC URL | Bundler URL | Explorer |
|---------|----------|---------|-------------|----------|
| Mainnet | 10088 | `https://gatelayer-mainnet.gatenode.cc` | Same as RPC (ERC-4337 methods) | https://www.gatescan.org/gatelayer |
| Testnet | 10087 | `https://gatelayer-testnet.gatenode.cc` | Same as RPC (ERC-4337 methods) | https://www.gatescan.org/gatelayer-testnet |

## Quick Start

### Installation

```bash
npm install @aa-sdk/core viem
```

### Basic Setup

```typescript
import {
  createSmartAccountClient,
  createBundlerClient,
  LocalAccountSigner,
} from "@aa-sdk/core";
import { http } from "viem";
import { gatelayerMainnet } from "./references/chains";

// 1. Create a signer
const signer = LocalAccountSigner.privateKeyToAccountSigner(
  process.env.PRIVATE_KEY as `0x${string}`
);

// 2. Create a smart account (Light Account)
// Note: You'll need a deployed Light Account factory on GateLayer
const smartAccount = await createLightAccount({
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  chain: gatelayerMainnet,
  signer,
});

// 3. Create smart account client
const client = createSmartAccountClient({
  chain: gatelayerMainnet,
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  account: smartAccount,
});

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

// Wait for confirmation
const receipt = await client.waitForUserOperationTransaction({
  hash: result.hash,
});

console.log("Transaction Receipt:", receipt);
```
```

**Step 2: Run: `git diff skills/building-with-gatelayer-aa/SKILL.md`**

Expected: Shows updated header and Quick Start section

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: update SKILL.md to use @aa-sdk-core"
```

---

### Task 7: Add chains.ts reference to skill

**Files:**
- Create: `skills/building-with-gatelayer-aa/references/chains.ts`
- Modify: `skills/building-with-gatelayer-aa/SKILL.md` (add reference link)

**Step 1: Copy chains.ts from test project**

```bash
cp ../aa-skill-test/src/config/chains.ts skills/building-with-gatelayer-aa/references/chains.ts
```

**Step 2: Update SKILL.md Feature References table**

Add row to table:
```markdown
| Chain Config | [references/chains.ts](references/chains.ts) | GateLayer Mainnet and Testnet chain definitions |
```

**Step 3: Test the file compiles**

Run: `cd skills/building-with-gatelayer-aa/references && npx tsc --noEmit chains.ts`

Expected: No errors

**Step 4: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/chains.ts skills/building-with-gatelayer-aa/SKILL.md
git commit -m "feat: add GateLayer chain configuration reference"
```

---

### Task 8: Update smart-wallet.md reference

**Files:**
- Modify: `skills/building-with-gatelayer-aa/references/smart-wallet.md:1-145`

**Step 1: Update account creation section**

Replace the "Creating an Account" section:

```markdown
## Creating an Account

### Basic Setup

```typescript
import {
  createSmartAccountClient,
  LocalAccountSigner,
} from "@aa-sdk/core";
import { http } from "viem";
import { gatelayerMainnet } from "./references/chains";

// 1. Create a signer from private key
const signer = LocalAccountSigner.privateKeyToAccountSigner(
  process.env.PRIVATE_KEY as `0x${string}`
);

// 2. Create Light Account
// Note: Requires deployed Light Account factory on GateLayer
import { createLightAccount } from "@aa-sdk/core";

const smartAccount = await createLightAccount({
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  chain: gatelayerMainnet,
  signer,
});

// 3. Create client
const client = createSmartAccountClient({
  chain: gatelayerMainnet,
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  account: smartAccount,
});

console.log("Account Address:", client.account.address);
```

### Import Existing Account

```typescript
// Import existing Light Account by address
const existingAccount = await createLightAccount({
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  chain: gatelayerMainnet,
  signer,
  accountAddress: "0xExistingLightAccount",
});

const client = createSmartAccountClient({
  chain: gatelayerMainnet,
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  account: existingAccount,
});
```
```

**Step 2: Test markdown syntax**

Run: `npx markdownlint skills/building-with-gatelayer-aa/references/smart-wallet.md 2>&1 || true`

Expected: No blocking errors

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/smart-wallet.md
git commit -m "docs: update smart-wallet.md for @aa-sdk/core"
```

---

### Task 9: Update batch-transactions.md reference

**Files:**
- Modify: `skills/building-with-gatelayer-aa/references/batch-transactions.md:30-110`

**Step 1: Update Execution Modes section**

Replace with @aa-sdk/core examples:

```markdown
## Execution Modes

### 1. Atomic Batch (Recommended)

All calls succeed or all revert together:

```typescript
const result = await client.sendUserOperation({
  uo: [
    { target: "0xToken", data: approveCall, value: 0n },
    { target: "0xDex", data: swapCall, value: 0n },
  ],
});

// All calls execute atomically in single UserOperation
```

**Use when**: Operations depend on each other (approve + swap)

### 2. Build Batch Before Sending

```typescript
import { buildUserOp } from "@aa-sdk/core";

const batchUo = await buildUserOp(client, [
  { target: "0xA", data: callA, value: 0n },
  { target: "0xB", data: callB, value: 0n },
  { target: "0xC", data: callC, value: 0n },
]);

const result = await client.sendUserOperation({
  uo: batchUo,
});
```
```

**Step 2: Update "Building Batches" section examples to use `@aa-sdk/core` patterns**

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/batch-transactions.md
git commit -m "docs: update batch-transactions.md for @aa-sdk/core"
```

---

### Task 10: Update authentication.md reference

**Files:**
- Modify: `skills/building-with-gatelayer-aa/references/authentication.md`

**Step 1: Update signer creation section**

Replace Alchemy-specific signer examples with @aa-sdk/core patterns:

```typescript
import { LocalAccountSigner } from "@aa-sdk/core";

// From private key
const signer = LocalAccountSigner.privateKeyToAccountSigner(
  process.env.PRIVATE_KEY as `0x${string}`
);

// From mnemonic
const signer = LocalAccountSigner.mnemonicToAccountSigner(
  process.env.MNEMONIC
);
```

**Step 2: Remove Alchemy OAuth references**

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/authentication.md
git commit -m "docs: update authentication.md for @aa-sdk/core"
```

---

### Task 11: Update paymaster.md reference

**Files:**
- Modify: `skills/building-with-gatelayer-aa/references/paymaster.md`

**Step 1: Add clear warning about Alchemy Gas Manager**

Add at top of file:

```markdown
> **IMPORTANT:** Alchemy Gas Manager is NOT available when using @aa-sdk/core with GateLayer's official infrastructure. This document describes paymaster integration for custom paymaster solutions.
```

**Step 2: Update paymaster middleware example**

```typescript
// Custom paymaster middleware for GateLayer
import { createPaymasterMiddleware } from "@aa-sdk/core";

const paymasterMiddleware = createPaymasterMiddleware({
  paymasterUrl: process.env.GATELAYER_PAYMASTER_URL,
  policy: {
    // Your sponsorship policy
  },
});

const client = createSmartAccountClient({
  chain: gatelayerMainnet,
  transport: http(rpcUrl),
  account: smartAccount,
  middleware: [paymasterMiddleware],
});
```

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/paymaster.md
git commit -m "docs: update paymaster.md for custom paymaster solutions"
```

---

### Task 12: Update troubleshooting.md reference

**Files:**
- Modify: `skills/building-with-gatelayer-aa/references/troubleshooting.md`

**Step 1: Add GateLayer-specific issues**

Add new section:

```markdown
## GateLayer-Specific Issues

### Bundler Methods Not Available

Verify GateLayer RPC supports ERC-4337 methods:

```bash
curl -X POST https://gatelayer-testnet.gatenode.cc \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_supportedEntryPoints","params":[],"id":1}'
```

Expected: Returns `["0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789"]`

### Account Factory Not Deployed

Error: "Smart account factory not yet deployed to GateLayer"

**Solution:** Deploy Light Account factory or use GateLayer's native solution if available.
```

**Step 2: Remove Alchemy-specific troubleshooting**

**Step 3: Commit**

```bash
git add skills/building-with-gatelayer-aa/references/troubleshooting.md
git commit -m "docs: update troubleshooting.md for GateLayer infrastructure"
```

---

### Task 13: Update Critical Requirements section in SKILL.md

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md:75-145`

**Step 1: Replace Critical Requirements section**

```markdown
## Critical Requirements

### Network Configuration

- **Use references/chains.ts** for correct chain definitions
- **Mainnet Chain ID**: 10088
- **Testnet Chain ID**: 10087
- **Official RPC**: `https://gatelayer-mainnet.gatenode.cc` / `https://gatelayer-testnet.gatenode.cc`
- **Bundler**: Same as RPC (supports ERC-4337 methods)
- **Always validate Chain ID** before signing transactions

### Account Factory Deployment

- **Light Account factory must be deployed** to GateLayer before use
- **Entry Point v0.6.0**: `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789`
- **Check factory address** matches GateLayer deployment
- **Test on testnet first** before mainnet deployment

### Gas Sponsorship

- **Alchemy Gas Manager NOT available** - use GateLayer Paymaster or self-sponsored
- **Custom paymaster** requires ERC-7677 compliance
- **Always estimate gas** before sending transactions
- **Provide fallback** to user-paid gas when sponsorship unavailable

### Security

- **Never expose private keys** - use environment variables
- **Validate all addresses** before sending transactions
- **Check deployment status** before assuming account exists
- **Use exact amounts** - avoid floating point math
- **Test on testnet** before mainnet transactions
```

**Step 2: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "docs: update Critical Requirements for GateLayer"
```

---

### Task 14: Update Architecture Notes section

**Files:**
- Modify: `skills/building-with-gatelayer-aa/SKILL.md:115-145`

**Step 1: Replace Architecture Notes**

```markdown
## Architecture Notes

### @aa-sdk/core vs Alchemy AA SDK

- **@aa-sdk/core** is open-source, no API key required
- **Uses GateLayer's official infrastructure** - RPC and Bundler
- **Full ERC-4337 compliance** - works with any bundler
- **No vendor lock-in** - can switch to any bundler provider

### Why @aa-sdk/core?

- **No API keys** - Use GateLayer directly
- **Open source** - MIT license, community maintained
- **Standard compliance** - Full ERC-4337 and ERC-1271 support
- **Multi-chain** - Works across all EVM chains
- **Flexible** - Can use any paymaster solution

### Bundler Configuration

GateLayer's RPC endpoint supports ERC-4337 methods:
- UserOperations are submitted to GateLayer's Bundler
- Bundler URL: Same as RPC
- No extra configuration needed
```

**Step 2: Commit**

```bash
git add skills/building-with-gatelayer-aa/SKILL.md
git commit -m "docs: update Architecture Notes for @aa-sdk/core"
```

---

## Phase 3: Create Test Files

### Task 15: Create test utilities

**Files:**
- Create: `../aa-skill-test/src/utils/setup.ts`

**Step 1: Create setup.ts**

```typescript
import {
  createSmartAccountClient,
  createBundlerClient,
  LocalAccountSigner,
} from "@aa-sdk/core";
import { http, createWalletClient } from "viem";
import { gatelayerTestnet } from "../config/chains";

export async function setupTest() {
  const privateKey = process.env.PRIVATE_KEY as `0x${string}`;
  if (!privateKey) {
    throw new Error("PRIVATE_KEY not set in .env");
  }

  const rpcUrl = process.env.GATELAYER_TESTNET_RPC || "https://gatelayer-testnet.gatenode.cc";

  // Create signer
  const signer = LocalAccountSigner.privateKeyToAccountSigner(privateKey);

  // Get EOA address for funding
  const walletClient = createWalletClient({
    account: signer.account,
    chain: gatelayerTestnet,
    transport: http(rpcUrl),
  });

  console.log("EOA Address:", signer.account.address);
  console.log("RPC URL:", rpcUrl);
  console.log("Chain ID:", gatelayerTestnet.id);

  return {
    signer,
    walletClient,
    chain: gatelayerTestnet,
    rpcUrl,
  };
}

export async function checkBalance(address: string) {
  const bundlerClient = createBundlerClient({
    chain: gatelayerTestnet,
    transport: http(process.env.GATELAYER_TESTNET_RPC || "https://gatelayer-testnet.gatenode.cc"),
  });

  const balance = await bundlerClient.getBalance({
    address: address as `0x${string}`,
  });

  console.log("Balance:", `${balance} wei`);

  return balance;
}
```

**Step 2: No commit needed**

---

### Task 16: Create test 01 - Create Account

**Files:**
- Create: `../aa-skill-test/src/tests/01-create-account.ts`

**Step 1: Create test file**

```typescript
import { setupTest, checkBalance } from "../utils/setup.js";
import { createSmartAccountClient } from "@aa-sdk/core";
import { http } from "viem";

async function test() {
  console.log("\n=== Test 01: Create Smart Account ===\n");

  // Setup
  const { signer, chain, rpcUrl } = await setupTest();

  // Check EOA balance
  const balance = await checkBalance(signer.account.address);
  if (balance === 0n) {
    console.error("ERROR: EOA has no balance. Please fund with testnet GT.");
    process.exit(1);
  }

  // TODO: Create Light Account when factory is deployed
  // For now, just verify the setup works

  console.log("\n✅ Test 01: Setup successful");
  console.log("\nNote: Light Account factory deployment required");
  console.log("See references/chains.ts for factory configuration\n");
}

test().catch(console.error);
```

**Step 2: Run test**

Run: `cd ../aa-skill-test && bun run test:01`

Expected: Setup successful message

**Step 3: No commit needed**

---

### Task 17: Create test 02 - Send Transaction

**Files:**
- Create: `../aa-skill-test/src/tests/02-send-tx.ts`

**Step 1: Create test file**

```typescript
import { setupTest } from "../utils/setup.js";
import { createSmartAccountClient } from "@aa-sdk/core";
import { http, parseEther } from "viem";

async function test() {
  console.log("\n=== Test 02: Send Transaction ===\n");

  const { signer, chain, rpcUrl } = await setupTest();

  const recipient = process.env.TEST_RECIPIENT as `0x${string}`;
  if (!recipient) {
    console.error("ERROR: TEST_RECIPIENT not set in .env");
    process.exit(1);
  }

  console.log("Recipient:", recipient);
  console.log("Amount: 0.001 GT");

  // TODO: Implement after Light Account factory deployment
  console.log("\nNote: Requires deployed Light Account factory\n");
}

test().catch(console.error);
```

**Step 2: No commit needed**

---

### Task 18: Create test 03 - Batch Transactions

**Files:**
- Create: `../aa-skill-test/src/tests/03-batch.ts`

**Step 1: Create test file**

```typescript
import { setupTest } from "../utils/setup.js";
import { createSmartAccountClient } from "@aa-sdk/core";
import { http } from "viem";

async function test() {
  console.log("\n=== Test 03: Batch Transactions ===\n");

  const { signer, chain, rpcUrl } = await setupTest();

  // TODO: Implement batch transaction test
  console.log("\nNote: Requires deployed Light Account factory\n");
}

test().catch(console.error);
```

**Step 2: No commit needed**

---

### Task 19: Create test 04 - Paymaster

**Files:**
- Create: `../aa-skill-test/src/tests/04-paymaster.ts`

**Step 1: Create test file**

```typescript
import { setupTest } from "../utils/setup.js";

async function test() {
  console.log("\n=== Test 04: Paymaster ===\n");

  const { signer } = await setupTest();

  console.log("\nNote: GateLayer Paymaster integration");
  console.log("Check if GateLayer provides paymaster service\n");
}

test().catch(console.error);
```

**Step 2: No commit needed**

---

## Phase 4: Verification

### Task 20: Verify GateLayer Testnet connectivity

**Files:**
- None (verification step)

**Step 1: Test RPC connection**

Run: `curl -X POST https://gatelayer-testnet.gatenode.cc -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}'`

Expected: `{"jsonrpc":"2.0","id":1,"result":"0x2767"}` (10087 in hex)

**Step 2: Test ERC-4337 support**

Run: `curl -X POST https://gatelayer-testnet.gatenode.cc -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"eth_supportedEntryPoints","params":[],"id":1}'`

Expected: Entry point address

**Step 3: Document results**

Create notes in README about testnet status

**Step 4: No commit needed**

---

### Task 21: Final verification and documentation

**Files:**
- Create: `../aa-skill-test/README.md`

**Step 1: Create test README**

```markdown
# AA Skill Test Project

Local test project for verifying GateLayer AA Wallet skill.

## Setup

1. Copy `.env.example` to `.env`
2. Add your private key
3. Fund your EOA with testnet GT

## Run Tests

```bash
bun run test:01  # Create account
bun run test:02  # Send transaction
bun run test:03  # Batch transactions
bun run test:04  # Paymaster
```

## Notes

- Light Account factory must be deployed to GateLayer
- Check GateLayer documentation for factory address
- Testnet faucet: [GateLayer Faucet]
```

**Step 2: No commit needed**

---

### Task 22: Final commit and summary

**Files:**
- Modify: `README.md` (root project)

**Step 1: Update main README with note about migration**

**Step 2: Final commit of all skill changes**

```bash
git add skills/building-with-gatelayer-aa/
git commit -m "feat: complete AA SDK migration to @aa-sdk/core

- Update all skill files to use @aa-sdk/core
- Add GateLayer chain configuration
- Remove Alchemy-specific dependencies
- Update all code examples and documentation
- Add comprehensive troubleshooting guide"
```

**Step 3: Create summary document**

Write summary of what was changed and verification steps

---

## Verification Checklist

After completing all tasks:

- [ ] All skill files updated with @aa-sdk/core examples
- [ ] `references/chains.ts` created with GateLayer configs
- [ ] Test project runs without errors
- [ ] GateLayer Testnet RPC is accessible
- [ ] ERC-4337 methods are supported
- [ ] Documentation is consistent across all files
- [ ] All commits follow conventional commits format

## Known Limitations

Document in skill:

- Light Account factory deployment status on GateLayer
- Alchemy Gas Manager unavailable
- Email/Social auth requires additional setup
- Paymaster depends on GateLayer infrastructure

## Next Steps

After implementation:

1. Deploy Light Account factory to GateLayer if needed
2. Complete test implementations
3. Run full integration tests
4. Update with actual factory addresses
5. Add more examples based on real testing
