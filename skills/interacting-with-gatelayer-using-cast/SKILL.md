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

## Call Contract Functions (Read-Only)

Call view/pure functions without sending transactions.

### Basic Syntax

```bash
cast call <contract_address> "<function_signature>" [args...] --rpc-url <rpc_url>
```

### Examples

Get token total supply:
```bash
cast call 0xA0b86a33E6441e88C5F2712C3E9b74B6F3f5a8b8 "totalSupply()(uint256)" --rpc-url https://gatelayer-mainnet.gatenode.cc
```

Get token decimals:
```bash
cast call 0xA0b86a33E6441e88C5F2712C3E9b74B6F3f5a8b8 "decimals()(uint8)" --rpc-url https://gatelayer-mainnet.gatenode.cc
```

Get contract owner:
```bash
cast call <contract_address> "owner()(address)" --rpc-url https://gatelayer-mainnet.gatenode.cc
```

Query a mapping with specific key:
```bash
cast call <contract_address> "balances(address)(uint256)" 0xYourAddress --rpc-url https://gatelayer-mainnet.gatenode.cc
```

## Send Transactions

Send GT or execute contract functions with proper safety checks.

### Safety Workflow

Always follow these steps before sending transactions on GateLayer:

1. **Verify chain ID** - Ensure you're on the correct network
2. **Check GT balance** - Confirm sufficient funds for gas + value
3. **Estimate gas** - Verify transaction will succeed
4. **Send transaction** - Execute with proper parameters

### Send GT (Native Token)

**Step 1: Verify Network**

```bash
cast chain-id --rpc-url https://gatelayer-mainnet.gatenode.cc
# Expected output: 10088
```

**Step 2: Check Balance**

```bash
cast balance 0xYourAddress --rpc-url https://gatelayer-mainnet.gatenode.cc
```

**Step 3: Estimate Gas**

```bash
cast estimate 0xRecipientAddress --rpc-url https://gatelayer-mainnet.gatenode.cc --value 1ether
```

**Step 4: Send Transaction**

```bash
cast send \
  --private-key $PRIVATE_KEY \
  --rpc-url https://gatelayer-mainnet.gatenode.cc \
  0xRecipientAddress \
  --value 1000000000000000000
```

This sends 1 GT (1 GT = 10^18 wei).

### Execute Contract Function

**Step 1: Estimate Gas**

```bash
cast estimate <contract_address> "<function_signature>" [args...] --rpc-url https://gatelayer-mainnet.gatenode.cc
```

**Step 2: Send Transaction**

```bash
cast send \
  --private-key $PRIVATE_KEY \
  --rpc-url https://gatelayer-mainnet.gatenode.cc \
  <contract_address> \
  "<function_signature>" \
  [args...]
```

Example - Approve ERC20 spend:
```bash
cast send \
  --private-key $PRIVATE_KEY \
  --rpc-url https://gatelayer-mainnet.gatenode.cc \
  0xA0b86a33E6441e88C5F2712C3E9b74B6F3f5a8b8 \
  "approve(address,uint256)" \
  0xSpenderAddress \
  1000000000000000000
```

## Blockchain Inspection

### Get Latest Block Number

**Mainnet:**
```bash
cast block-number --rpc-url https://gatelayer-mainnet.gatenode.cc
```

**Testnet:**
```bash
cast block-number --rpc-url https://gatelayer-testnet.gatenode.cc
```

### Get Block Details

Get full block information:
```bash
cast block <block_number> --rpc-url <rpc_url>
```

Example - Get specific block on Mainnet:
```bash
cast block 1234567 --rpc-url https://gatelayer-mainnet.gatenode.cc
```

Get latest block:
```bash
cast block --latest --rpc-url https://gatelayer-mainnet.gatenode.cc
```

### Get Transaction Details

Get transaction information:
```bash
cast tx <tx_hash> --rpc-url <rpc_url>
```

Example - Get transaction on Mainnet:
```bash
cast tx 0xabc123def456... --rpc-url https://gatelayer-mainnet.gatenode.cc
```

### Verify Network Connection

Check which GateLayer network you're connected to:

**Mainnet:**
```bash
cast chain-id --rpc-url https://gatelayer-mainnet.gatenode.cc
# Expected: 10088
```

**Testnet:**
```bash
cast chain-id --rpc-url https://gatelayer-testnet.gatenode.cc
# Expected: 10087
```

## Error Handling & Troubleshooting

### Common Errors

#### RPC Connection Errors

**Symptoms:** `Failed to fetch`, `Connection refused`, `timeout`

**Solutions:**
- Check RPC URL is correct: `https://gatelayer-mainnet.gatenode.cc` or `https://gatelayer-testnet.gatenode.cc`
- Verify network connectivity: `ping gatelayer-mainnet.gatenode.cc`
- Try alternative: use a self-hosted node (see @running-a-gatelayer-node)

#### Insufficient GT Balance

**Symptoms:** `insufficient funds for gas * price + value`

**Solutions:**
- Check balance: `cast balance <address> --rpc-url <rpc_url>`
- Ensure you have enough GT for gas + transaction amount
- For testnet: get testnet GT from GateLayer faucets
- For mainnet: acquire GT through bridges or exchanges

#### Gas Estimation Failed

**Symptoms:** `gas required exceeds allowance`, `execution reverted`

**Solutions:**
- Verify contract address exists on GateLayer
- Check function signature is correct
- Ensure contract is deployed on the network you're using
- Use `cast estimate` before sending to catch errors early

#### Invalid Chain ID

**Symptoms:** `chain ID mismatch`, transaction fails silently

**Solutions:**
- Verify chain ID: `cast chain-id --rpc-url <rpc_url>`
- Mainnet should return: `10088`
- Testnet should return: `10087`
- Ensure wallet/tools are configured for correct network

#### Private Key Errors

**Symptoms:** `invalid private key`, `failed to decrypt`, `invalid length`

**Solutions:**
- Ensure private key is 0x-prefixed hex string (64 hex characters + 0x)
- Never expose private keys in scripts, logs, or version control
- Use environment variables: `export PRIVATE_KEY=0x...`

### Transaction Safety Checklist

Before sending transactions on GateLayer:

- [ ] Verify chain ID (10088 for mainnet, 10087 for testnet)
- [ ] Check sufficient GT balance for gas + transaction value
- [ ] Estimate gas before sending
- [ ] Verify recipient address is correct
- [ ] Test on testnet first for new contracts or large amounts
- [ ] Use small test transactions before large transfers
