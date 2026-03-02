---
name: interacting-with-gatelayer-using-cast
description: Interact with GateLayer blockchain using Foundry's cast tool for querying GT balances, calling contracts, sending transactions, and blockchain exploration. Use when needing to interact with GateLayer Mainnet or Testnet via command-line, including reading contract state, sending GT, executing contract functions, or inspecting blockchain data. Covers phrases like "GateLayer cast balance", "query GT balance", "send GT on GateLayer", "cast GateLayer contract", "GateLayer block explorer", or "check GateLayer transaction".
---

# Interacting with GateLayer Using Cast

This skill enables interaction with GateLayer blockchain through Foundry's `cast` command-line tool. It covers common blockchain operations like GT balance queries, contract calls, transaction sending with safety checks, and network inspection.

## Overview

Use this skill when you need to:
- Query GT balances on GateLayer Mainnet or Testnet
- Call read-only contract functions on GateLayer
- Send GT or execute contract transactions
- Inspect blocks, transactions, and network state
- Estimate gas and verify transactions before sending

## Installation

To use this skill, you need Foundry installed, which provides the `cast` command.

Follow the official installation guide: https://getfoundry.sh/introduction/installation

### Quick Install (Linux/Mac)

1. Install Foundryup:
   ```bash
   curl -L https://foundry.paradigm.xyz | bash
   ```

2. Source your shell configuration or start a new terminal:
   ```bash
   source ~/.bashrc  # or ~/.zshrc
   ```

3. Install Foundry:
   ```bash
   foundryup
   ```

4. Verify installation:
   ```bash
   cast --version
   ```

For other platforms or detailed instructions, see the full guide at https://getfoundry.sh/introduction/installation

## Prerequisites

- Foundry installed (`cast` command available)
- GateLayer RPC endpoint (Mainnet or Testnet - provided below)
- Private key for transaction operations (if sending GT or writing to contracts)

## GateLayer Network Configuration

This skill uses hardcoded GateLayer RPC endpoints for quick access. For advanced setup or alternative RPC providers, see the @connecting-to-gatelayer-network skill.

### Mainnet

| Property | Value |
|----------|-------|
| Network Name | GateLayer |
| Chain ID | `10088` |
| RPC Endpoint | `https://gatelayer-mainnet.gatenode.cc` |
| Native Currency | **GT** |
| Explorer | https://www.gatescan.org/gatelayer |

### Testnet

| Property | Value |
|----------|-------|
| Network Name | GateLayer Testnet |
| Chain ID | `10087` |
| RPC Endpoint | `https://gatelayer-testnet.gatenode.cc` |
| Native Currency | **GT** |
| Explorer | https://www.gatescan.org/gatelayer-testnet |

### Quick Reference

**Mainnet RPC**: `https://gatelayer-mainnet.gatenode.cc`
**Testnet RPC**: `https://gatelayer-testnet.gatenode.cc`
**Mainnet Chain ID**: `10088`
**Testnet Chain ID**: `10087`

## Query GT Balance

Get native GT balance of an address on GateLayer.

### Mainnet

```bash
cast balance <address> --rpc-url https://gatelayer-mainnet.gatenode.cc
```

### Testnet

```bash
cast balance <address> --rpc-url https://gatelayer-testnet.gatenode.cc
```

### Example

Check GT balance on Mainnet:
```bash
cast balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url https://gatelayer-mainnet.gatenode.cc
# Output: 1.5 GT
```

Check GT balance on Testnet:
```bash
cast balance 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url https://gatelayer-testnet.gatenode.cc
# Output: 10 GT (example)
```

### Query ERC20 Token Balance

Query ERC20 token balances on GateLayer:

```bash
cast call <token_contract> "balanceOf(address)(uint256)" <address> --rpc-url <rpc_url>
```

Example:
```bash
cast call 0xA0b86a33E6441e88C5F2712C3E9b74B6F3f5a8b8 "balanceOf(address)(uint256)" 0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045 --rpc-url https://gatelayer-mainnet.gatenode.cc
```
