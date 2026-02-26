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
