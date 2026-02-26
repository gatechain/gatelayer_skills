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
