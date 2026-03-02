# Interacting with GateLayer Using Cast - Design Document

**Date**: 2025-03-02
**Status**: Approved
**Skill Name**: `interacting-with-gatelayer-using-cast`

## Overview

A GateLayer-specific skill for interacting with the GateLayer blockchain using Foundry's `cast` command-line tool. The skill enables AI agents to query GT balances, call contracts, send transactions, and inspect blockchain data on both GateLayer Mainnet and Testnet.

## Design Decisions

### Network Support
- **Both Mainnet and Testnet**: Full support for GateLayer Mainnet (Chain ID: 10088) and Testnet (Chain ID: 10087)
- Clear distinction between networks in all examples

### RPC Configuration
- **Hardcoded official RPC URLs**: Directly embed GateLayer's official endpoints for convenience
  - Mainnet: `https://gatelayer-mainnet.gatenode.cc`
  - Testnet: `https://gatelayer-testnet.gatenode.cc`
- Reference `connecting-to-gatelayer-network` skill for advanced setup

### Token Focus
- **GT as native token**: Emphasize GT as GateLayer's native token (like ETH on Ethereum)
- Primary balance queries use native `cast balance` command
- ERC20 queries treated as secondary operations

### Safety Level
- **Standard safety checks** included:
  - Balance verification before transactions
  - Gas estimation before sending
  - Chain ID verification
  - Testnet-first workflow guidance

### Feature Scope
**Standard feature set** includes:
- GT balance queries
- Contract calls (read-only)
- Transaction sending with safety checks
- Blockchain inspection (blocks, transactions)
- Data decoding
- Gas price queries
- Batch operations

### GateLayer-Native Experience
- Official GateLayer network configuration
- GT token symbol throughout examples
- GateLayer Mainnet/Testnet chain IDs
- Reference to GateScan explorer
- Cross-references to other GateLayer skills

## Skill Structure

### 1. Overview & Metadata
- Skill name and description
- Purpose and use cases
- Trigger phrases for AI agents

### 2. Installation & Prerequisites
- Foundry installation instructions
- System requirements
- Verification commands

### 3. GateLayer Network Configuration
- Mainnet configuration table (Chain ID: 10088, RPC, GT token)
- Testnet configuration table (Chain ID: 10087, RPC, GT token)
- Hardcoded RPC URLs for quick usage

### 4. Common Operations

#### 4.1 Query GT Balance
- Native GT balance queries for both networks
- Example commands with output

#### 4.2 Query ERC20 Token Balance
- ERC20 balance queries on GateLayer
- Function signature examples

#### 4.3 Call Contract Functions (Read-Only)
- View/pure function calls
- Examples with common operations (total supply, etc.)

#### 4.4 Send Transactions
- Step-by-step workflow:
  1. Check GT balance
  2. Estimate gas
  3. Send transaction
- Safety checks at each step

### 5. Blockchain Inspection
- Get latest block number
- Get block details
- Get transaction details
- Verify chain ID

### 6. Error Handling & Troubleshooting
- Common errors and solutions:
  - RPC connection errors
  - Insufficient GT balance
  - Gas estimation failures
  - Invalid chain ID
  - Private key errors
- Safety checklist for transactions

### 7. Advanced Usage
- Data decoding (hex to ASCII, decimal, wei to GT)
- Batch operations (shell loops)
- Gas price queries
- Contract deployment with cast
- Working with ABIs

### 8. GateLayer-Specific Examples
- Real-world examples for:
  - Checking GT balance on Mainnet/Testnet
  - Sending GT with safety checks
  - Querying GateLayer blocks
  - Verifying network connection

## Technical Implementation

### File Structure
```
skills/
  └── interacting-with-gatelayer-using-cast/
      └── SKILL.md
```

### Dependencies
- Foundry (cast command)
- RPC endpoints (hardcoded)
- Optional: Private keys for transactions

### Cross-Skill References
- `connecting-to-gatelayer-network`: Detailed network configuration
- `deploying-contracts-on-gatelayer`: Complex contract deployments
- `running-a-gatelayer-node`: Self-hosted RPC infrastructure

## Implementation Notes

1. **Single File Design**: All content in one SKILL.md file for simplicity and maintainability
2. **Example-Driven**: Every section includes practical, copy-pasteable examples
3. **Safety-First**: Transaction workflows include verification steps
4. **Network-Aware**: Clear distinction between Mainnet and Testnet throughout

## Success Criteria

- [ ] Users can query GT balances on both GateLayer networks
- [ ] Users can call read-only contract functions
- [ ] Users can send GT with proper safety checks
- [ ] Users can inspect blocks and transactions
- [ ] Error handling covers common issues
- [ ] GateLayer-specific examples work as documented
- [ ] Skill follows existing repo structure and patterns

## Next Steps

1. Create `skills/interacting-with-gatelayer-using-cast/SKILL.md`
2. Implement content following this design
3. Test examples on GateLayer Testnet
4. Update README.md to include new skill
5. Commit and create PR (if needed)
