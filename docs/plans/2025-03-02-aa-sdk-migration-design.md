# AA SDK Migration Design

## Overview

Migrate the `building-with-gatelayer-aa` skill from Alchemy's proprietary AA SDK to the open-source `@aa-sdk/core`, enabling use of GateLayer's official RPC and Bundler infrastructure without requiring Alchemy API Keys.

## Project Goals

1. **Migrate to `@aa-sdk/core`** - Replace `@alchemy/aa-alchemy` with the open-source AA SDK
2. **Use official GateLayer infrastructure** - Official RPC and Bundler, no external dependencies
3. **Create local test project** - Verify the updated skill works correctly on GateLayer Testnet

## Architecture Changes

### Current State

- **SDK**: `@alchemy/aa-alchemy`
- **Requirement**: Alchemy API Key
- **Bundler**: Alchemy's Bundler (automatic)
- **Paymaster**: Alchemy Gas Manager (not available on GateLayer)

### New State

- **SDK**: `@aa-sdk/core` (open-source)
- **Requirement**: None (uses GateLayer directly)
- **Bundler**: `https://gatelayer-bundler.gatenode.cc`
- **Paymaster**: GateLayer's Paymaster (if available) or self-sponsored

## Network Configuration

| Network | Chain ID | RPC URL | Bundler URL | Explorer |
|---------|----------|---------|-------------|----------|
| Mainnet | 10088 | `https://gatelayer-mainnet.gatenode.cc` | Same as RPC (ERC-4337) | https://www.gatescan.org/gatelayer |
| Testnet | 10087 | `https://gatelayer-testnet.gatenode.cc` | Same as RPC (ERC-4337) | https://www.gatescan.org/gatelayer-testnet |

## File Modifications

### Core Skill Files

| File | Changes |
|------|---------|
| `skills/building-with-gatelayer-aa/SKILL.md` | Update dependencies, Quick Start, configuration |
| `references/smart-wallet.md` | Update account creation with `@aa-sdk/core` |
| `references/batch-transactions.md` | Update batch transaction examples |
| `references/authentication.md` | Update auth flow (remove Alchemy-specific) |
| `references/paymaster.md` | Update Paymaster docs (Alchemy Gas Manager unavailable) |
| `references/troubleshooting.md` | Update troubleshooting guide |

### New Files to Add

| File | Purpose |
|------|---------|
| `references/chains.ts` | GateLayer chain definitions |
| `examples/simple-transaction.ts` | Simple transaction example |
| `examples/advanced-usage.ts` | Advanced features (batch, sign, etc.) |

## Code Changes

### Dependency Update

**Before:**
```json
{
  "@alchemy/aa-alchemy": "^1.x.x",
  "@alchemy/aa-accounts": "^1.x.x",
  "@alchemy/aa-core": "^1.x.x"
}
```

**After:**
```json
{
  "@aa-sdk/core": "^1.x.x",
  "viem": "^2.x.x"
}
```

### Client Creation Example

**Before (Alchemy SDK):**
```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
  owner: signer,
});
```

**After (aa-sdk-core):**
```typescript
import { createSmartAccountClient, LocalAccountSigner } from "@aa-sdk/core";
import { http } from "viem";

const signer = LocalAccountSigner.privateKeyToAccountSigner(privateKey);

const client = createSmartAccountClient({
  chain: gatelayerMainnet,
  transport: http("https://gatelayer-mainnet.gatenode.cc"),
  account: smartAccount,
});
```

## Local Test Project

### Structure

```
/aa-skill-test  (NOT committed to git)
├── package.json
├── tsconfig.json
├── .env
├── .gitignore
├── src/
│   ├── config/
│   │   └── chains.ts          # GateLayer chain configs
│   ├── tests/
│   │   ├── 01-create-account.ts    # Wallet creation
│   │   ├── 02-send-tx.ts           # Send transaction
│   │   ├── 03-batch.ts             # Batch transactions
│   │   └── 04-paymaster.ts         # Paymaster testing
│   └── utils/
│       └── setup.ts            # Test utilities
└── README.md
```

### Test Coverage

1. **Wallet Creation** - Create smart account from private key
2. **Send Transaction** - Send GT to another address
3. **Batch Transactions** - Send multiple transactions atomically
4. **Paymaster** - Test gas sponsorship (if available)

### Test Execution

Interactive testing with step-by-step confirmation:
```bash
bun run test:01    # Create account
bun run test:02    # Send transaction
bun run test:03    # Batch transactions
bun run test:04    # Paymaster
```

## Constraints and Limitations

### Known Limitations

1. **Alchemy Gas Manager Unavailable** - Cannot use Alchemy's paymaster
2. **Account Factory** - Need to verify Light Account Factory is deployed on GateLayer
3. **Email/Social Auth** - Requires additional provider setup (testing with private key only)

### Mitigation Strategies

- If Light Account Factory is not deployed, deploy a temporary one for testing
- Check if GateLayer provides native account abstraction solution
- Document clearly that Alchemy-specific features are unavailable

## Entry Points

GateLayer supports standard ERC-4337 entry points:
- **v0.6.0**: `0x5FF137D4b0FDCD49DcA30c7CF57E578a026d2789`
- **v0.7.0**: Check GateLayer documentation

## References

- Alchemy AA SDK: https://docs.alchemy.com/reference/aa-sdk
- ERC-4337 Spec: https://eips.ethereum.org/EIPS/eip-4337
- GateLayer Docs: https://www.gatechain.io/docs/GateLayer/Introduction/
- Reference Implementation: `/Users/finn/.agents/skills/aa-sdk-gatelayer`

## Success Criteria

1. ✅ All skill files updated with `@aa-sdk/core` examples
2. ✅ Local test project created and functional
3. ✅ All test cases pass on GateLayer Testnet
4. ✅ Transactions visible on GateScan Testnet
5. ✅ Documentation is accurate and complete

## Timeline

- Design validation: ✅ Complete
- Implementation: Pending (see implementation plan)
- Testing: Pending
- Documentation: Pending
