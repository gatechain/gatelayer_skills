# GateLayer AA Wallet Skill Design

**Date**: 2025-02-25
**Status**: Approved
**Author**: GateLayer Development Team

## Overview

Create a comprehensive Agent Skill for integrating Alchemy Account Abstraction (AA) wallets on GateLayer network. This skill enables AI agents to help developers implement smart wallet functionality including multi-provider authentication, gas sponsorship, and batch transactions.

## Objectives

1. Enable AI agents to assist developers in building GateLayer applications with smart wallets
2. Provide comprehensive reference documentation for all AA wallet features
3. Follow Base Account skill structure for consistency and AI agent familiarity
4. Support both GateLayer Mainnet (Chain ID: 10088) and Testnet (Chain ID: 10087)

## Technical Stack

- **SDK**: Alchemy AA SDK (Light Account)
- **Language**: TypeScript/JavaScript
- **Network**: GateLayer (OP Stack L2)
- **Standard**: ERC-4337 (Account Abstraction)
- **Dependencies**: `connecting-to-gatelayer-network` skill for network configuration

## File Structure

```
skills/building-with-gatelayer-aa/
├── SKILL.md                           # Main entry point
└── references/
    ├── authentication.md              # Multi-provider authentication
    ├── smart-wallet.md                # Light Account management
    ├── paymaster.md                   # Gas sponsorship configuration
    ├── batch-transactions.md          # Batch transaction implementation
    └── troubleshooting.md             # Debugging and common issues
```

## Core Features

### 1. Smart Wallet (Light Account)

Based on Ethereum Foundation's SimpleAccount:
- Gas-optimized ERC-4337 smart contract account
- ERC-1271 signature support (OpenSea compatible)
- Ownership transfer capabilities
- Quantstamp-audited

**Key Benefits**:
- No seed phrases required
- Gasless transactions via Paymaster
- Multi-chain support

### 2. Multi-Provider Authentication

Supported authentication methods:
- **Email** - Magic.link style passwordless login
- **Social Login** - Google, Apple, Facebook
- **Passkeys** - WebAuthn biometric authentication
- **Traditional Wallets** - MetaMask, WalletConnect
- **MPC Wallets** - Privy, Turnkey, Dynamic

### 3. Gas Sponsorship (Paymaster)

Integration with multiple Paymaster providers:
- **Alchemy Gas Manager** - Official Alchemy solution
- **Pimlico** - ERC-4337 compatible Paymaster
- **Gelato Relay** - Decentralized Paymaster network
- **ZeroDev** - Open-source Paymaster

**Sponsorship Policies**:
- Whitelist mode (specific addresses only)
- Limit mode (daily/monthly caps)
- Transaction type filtering
- Global policies (use with caution)

### 4. Batch Transactions

Execute multiple operations in a single transaction:
- **Atomic Batching** - All succeed or all revert
- **Sequential Execution** - Execute in order, stop on error
- **Parallel Execution** - Independent execution

**Use Cases**:
- Token approve + swap (DeFi)
- Mint NFT + payment
- Multi-transfer (batch payments)
- Complex multi-step DeFi operations

## Content Structure

### Main SKILL.md

**Sections**:
1. Front matter (YAML metadata)
2. Quick Start Guide
3. Feature Reference Table
4. Critical Requirements
   - Security considerations
   - Network configuration
   - Popup handling
   - Light Account requirements
5. Architecture Notes
   - Alchemy AA SDK vs Base SDK
   - Light Account rationale
   - Bundler endpoint configuration
6. Links to detailed references

### Reference Documents

Each reference document follows this structure:
- Table of Contents
- Overview
- How It Works
- Code Examples
- Configuration Options
- Security Checklist
- Common Pitfalls

## Key Differences from Base Account

| Aspect | Base Account | GateLayer AA Wallet |
|--------|--------------|---------------------|
| SDK | `@base-org/account` | Alchemy AA SDK |
| Account Type | Custom Base Account | Light Account (standard) |
| Chain IDs | 8453 (main), 84532 (test) | 10088 (main), 10087 (test) |
| RPC Endpoints | Base RPC | GateLayer RPC |
| Paymaster | Built-in `paymasterService` | Alchemy Gas Manager API |
| Special Features | Coinbase MagicSpend | GateLayer-specific optimizations |

## Security Considerations

### Critical Security Measures

1. **Replay Attack Prevention**
   - Generate nonces before user clicks "Sign in"
   - Track used nonces server-side with unique constraints
   - Reject any reused nonce

2. **Signature Verification**
   - Verify signatures on backend, never trust frontend
   - Use viem's `verifyMessage`/`verifyTypedData`
   - Handles both EOA and smart wallet (ERC-6492) automatically

3. **Paymaster URL Protection**
   - Never expose Paymaster URLs client-side
   - Use proxy server to protect credentials
   - Ensure Paymaster providers are ERC-7677 compliant

4. **Identity Verification**
   - Verify sender matches authenticated user
   - Validate `amount` and `recipient` server-side
   - Check transaction IDs to prevent impersonation

5. **Popup Handling**
   - Use `Cross-Origin-Opener-Policy: same-origin-allow-popups`
   - Generate nonces before user interaction
   - `same-origin` breaks the wallet popup

## Implementation Notes

### Network Configuration

Reference `connecting-to-gatelayer-network` skill for:
- Chain IDs (Mainnet: 10088, Testnet: 10087)
- RPC endpoints
- Explorer URLs

### SDK Initialization Example

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";

const chain = {
  id: 10088, // From connecting-to-gatelayer-network
  name: "GateLayer",
  rpcUrls: {
    public: {
      http: ["https://gatelayer-mainnet.gatenode.cc"]
    }
  }
};

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain,
  owner: signer,
});
```

## Success Criteria

1. AI agents can successfully guide developers through:
   - Creating a Light Account on GateLayer
   - Implementing multi-provider authentication
   - Setting up gas sponsorship with Paymaster
   - Building batch transactions

2. All code examples are:
   - Written in TypeScript/JavaScript
   - Tested on GateLayer Testnet
   - Compatible with Alchemy AA SDK latest version

3. Documentation quality:
   - Matches Base Account structure
   - Clear, concise, and actionable
   - Includes security best practices
   - Covers common edge cases

## Future Enhancements (Out of Scope for V1)

- Modular Account support (ERC-6900)
- Session keys for repeated actions
- Advanced spending permissions
- Sub-accounts feature
- Paymaster policy templates
- GateLayer-specific optimizations

## References

- Alchemy AA SDK: https://docs.alchemy.com/reference/aa-sdk
- ERC-4337: https://eips.ethereum.org/EIPS/eip-4337
- Base Account Skills: https://github.com/base/skills
- GateLayer Docs: https://www.gatechain.io/docs/GateLayer/Introduction/

## Appendix: Trigger Phrases

The skill should activate when AI agents encounter phrases like:
- "create AA wallet on GateLayer"
- "GateLayer smart wallet"
- "Gasless transactions GateLayer"
- "Paymaster setup GateLayer"
- "batch transactions GateLayer"
- "account abstraction GateLayer"
- "GateLayer SIWB"
- "sponsor gas GateLayer"
- "Light Account GateLayer"
