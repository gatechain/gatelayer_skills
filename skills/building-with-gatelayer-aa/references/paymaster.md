# Gas Sponsorship (Paymaster)

## Table of Contents

- [Overview](#overview)
- [How Paymasters Work](#how-paymasters-work)
- [Supported Providers](#supported-providers)
- [Alchemy Gas Manager Setup](#alchemy-gas-manager-setup)
- [Sponsorship Policies](#sponsorship-policies)
- [Implementation](#implementation)
- [Cost Monitoring](#cost-monitoring)
- [Security Considerations](#security-considerations)
- [Troubleshooting](#troubleshooting)

## Overview

Paymasters enable gasless transactions by sponsoring user gas fees. Users can interact with your application without holding GT, while you cover the costs.

## How Paymasters Work

1. **User initiates transaction** (no GT required)
2. **Paymaster validates** request against policy
3. **Paymaster sponsors gas** for the transaction
4. **Transaction executes** user pays 0 gas
5. **You reimburse** Paymaster for sponsored gas

## Supported Providers

| Provider | ERC-7677 Compatible | Setup Complexity | Features |
|----------|-------------------|------------------|----------|
| Alchemy Gas Manager | ✅ | Low | Dashboard, policies, API |
| Pimlico | ✅ | Medium | Open-source, flexible |
| Gelato Relay | ✅ | Medium | Decentralized network |
| ZeroDev | ✅ | Medium | Open-source, cheap |

## Alchemy Gas Manager Setup

### 1. Create Policy

1. Go to [Alchemy Dashboard](https://dashboard.alchemy.com/)
2. Navigate to Gas Manager
3. Create new policy with rules
4. Copy Policy ID

### 2. Configure Client

```typescript
import { createLightAccountAlchemyClient } from "@alchemy/aa-alchemy";

const client = await createLightAccountAlchemyClient({
  apiKey: process.env.ALCHEMY_API_KEY,
  chain: gatelayerMainnet,
  owner: signer,
  gasManagerConfig: {
    policyId: process.env.GATELAYER_GAS_POLICY_ID,
  },
});
```

## Sponsorship Policies

### Policy Types

#### 1. Whitelist Mode (Recommended)

Only sponsor transactions from specific addresses:

```typescript
// Alchemy Dashboard Policy Rules
{
  "rules": [
    {
      "type": "whitelist",
      "addresses": [
        "0xVerifiedUser1",
        "0xVerifiedUser2"
      ]
    }
  ]
}
```

#### 2. Limit Mode

Sponsor up to X gas per address/day:

```typescript
{
  "rules": [
    {
      "type": "spendingLimit",
      "limit": "10000000000000000", // 0.01 GT per day
      "period": "daily"
    }
  ]
}
```

#### 3. Transaction Type Filter

Only sponsor specific contract calls:

```typescript
{
  "rules": [
    {
      "type": "contractCall",
      "allowedContracts": [
        "0xMyNFTContract",
        "0xMyTokenContract"
      ],
      "allowedFunctions": [
        "mint",
        "transfer"
      ]
    }
  ]
}
```

#### 4. Global Policy

Sponsor all transactions (use with caution):

```typescript
{
  "rules": [
    {
      "type": "all"
    }
  ]
}
```

**⚠️ Warning**: Global policies can lead to unlimited costs. Use limits.

## Implementation

### Send Sponsored Transaction

```typescript
const result = await client.sendUserOperation({
  uo: {
    target: "0xNFTContract",
    data: encodeFunctionData({
      abi: nftAbi,
      functionName: "mint",
      args: [recipientAddress],
    }),
    value: 0n,
  },
  // Gas sponsorship happens automatically
  // based on gasManagerConfig policy
});

console.log("Sponsored Transaction:", result.hash);
```

### Check Sponsorship Status

```typescript
const gasDetails = await client.estimateUserOperationGas({
  uo: {
    target: "0xContract",
    data: "0x",
    value: 0n,
  },
});

console.log("Gas to be sponsored:", gasDetails);
```

### Conditional Sponsorship

```typescript
// Only sponsor if policy allows
try {
  const result = await client.sendUserOperation({
    uo: { /* transaction */ },
  });
  console.log("Transaction sponsored");
} catch (error) {
  if (error.message.includes("paymaster")) {
    // Fallback: user pays gas
    console.log("Sponsorship unavailable, user pays gas");
    const result = await client.sendUserOperation({
      uo: { /* transaction */ },
      skipGasSponsorship: true,
    });
  }
}
```

## Cost Monitoring

### Alchemy Dashboard

Monitor in real-time:
- Total gas sponsored
- Cost per transaction
- Policy utilization
- Daily/monthly spending

### API Monitoring

```typescript
// Track sponsored transactions in your database
await db.sponsoredTransactions.create({
  data: {
    userAddress,
    transactionHash: result.hash,
    gasSponsored: gasEstimate,
    timestamp: new Date(),
  },
});

// Query for analytics
const totalSponsored = await db.sponsoredTransactions.aggregate({
  _sum: { gasSponsored: true },
  where: {
    timestamp: {
      gte: startOfMonth,
    },
  },
});
```

### Set Budget Alerts

```typescript
// Check budget before sponsoring
const monthlySpend = await getMonthlySpend();

if (monthlySpend >= MONTHLY_BUDGET) {
  throw new Error("Monthly gas budget exceeded");
}
```

## Security Considerations

### Protect Paymaster URL

**❌ NEVER expose in frontend:**

```typescript
// BAD: Paymaster URL visible in browser
const client = createClient({
  paymasterUrl: "https://paymaster.com?secret=KEY"
});
```

**✅ Use backend proxy:**

```typescript
// Frontend requests sponsorship
const response = await fetch("/api/sponsor-gas", {
  method: "POST",
  body: JSON.stringify({ userOperation }),
});

// Backend validates and sponsors
// Backend: server/app.ts
app.post("/api/sponsor-gas", async (req, res) => {
  const { userOperation } = req.body;

  // Validate request
  if (!isValidUserOperation(userOperation)) {
    return res.status(400).json({ error: "Invalid" });
  }

  // Backend sponsors with secret key
  const paymasterData = await alchemy.paymaster.sponsor({
    policyId: process.env.GAS_POLICY_ID,
    userOperation,
  });

  res.json({ paymasterData });
});
```

### Validate Policies

```typescript
// Backend: Enforce additional checks
app.post("/api/sponsor-gas", async (req, res) => {
  const { userAddress, targetContract } = req.body;

  // Check if user is whitelisted
  const isWhitelisted = await db.whitelist.findUnique({
    where: { address: userAddress },
  });

  if (!isWhitelisted) {
    return res.status(403).json({ error: "Not whitelisted" });
  }

  // Check if contract is allowed
  const allowedContracts = [
    "0xNFTContract",
    "0xTokenContract",
  ];

  if (!allowedContracts.includes(targetContract)) {
    return res.status(403).json({ error: "Contract not allowed" });
  }

  // Sponsor transaction
  // ...
});
```

### Prevent Abuse

```typescript
// Rate limiting per user
const sponsorCount = await redis.incr(`sponsor:${userAddress}:${today}`);

if (sponsorCount > MAX_DAILY_SPONSORED_TXS) {
  return res.status(429).json({ error: "Daily limit exceeded" });
}

// Detect unusual patterns
const recentTxCount = await getRecentTransactionCount(userAddress);

if (recentTxCount > SUSPICIOUS_THRESHOLD) {
  // Flag for review
  await flagSuspiciousActivity(userAddress);
  return res.status(403).json({ error: "Flagged for review" });
}
```

## Troubleshooting

### Paymaster Denied Transaction

**Error**: "Paymaster denied request"

**Causes**:
1. Address not whitelisted
2. Spending limit exceeded
3. Contract not allowed
4. Invalid Paymaster configuration

**Solution**:
```typescript
// Check policy status
const policyStatus = await alchemy.gasManager.getPolicyStatus({
  policyId: process.env.GAS_POLICY_ID,
});

console.log("Policy Status:", policyStatus);
```

### Gas Estimation Failed

**Error**: "Gas estimation failed"

**Solution**:
```typescript
// Try without sponsorship
const estimate = await client.estimateUserOperationGas({
  uo: { /* transaction */ },
  skipGasSponsorship: true,
});

// If this works, Paymaster is the issue
```

### Paymaster Timeout

**Error**: "Paymaster request timeout"

**Solution**:
```typescript
// Increase timeout
const client = createLightAccountAlchemyClient({
  // ...
  gasManagerConfig: {
    policyId: process.env.GAS_POLICY_ID,
    timeout: 30000, // 30 seconds
  },
});
```

## Best Practices

1. **Start with whitelist mode** - Only sponsor verified users
2. **Set spending limits** - Prevent unlimited costs
3. **Monitor daily** - Check Alchemy dashboard regularly
4. **Use fallback** - Allow user-paid gas when sponsorship unavailable
5. **Backend validation** - Never trust frontend requests alone
6. **Budget alerts** - Set up alerts for unusual spending
7. **Test on testnet** - Verify policies before mainnet
8. **Document policies** - Keep team informed of rules

## Cost Calculation

### Estimate Monthly Costs

```typescript
// Example: 1000 users, 5 tx/day each, 0.001 GT gas
const dailyTxs = 1000 * 5; // 5000
const monthlyTxs = dailyTxs * 30; // 150,000
const gasPerTx = parseEther("0.001"); // 0.001 GT
const monthlyCost = monthlyTxs * gasPerTx;

console.log("Estimated Monthly Gas Cost:", formatEther(monthlyCost), "GT");
// Output: "Estimated Monthly Gas Cost: 150 GT"
```

### Optimize Costs

- **Batch transactions** - Combine multiple operations
- **Use efficient contracts** - Lower gas functions
- **Set appropriate limits** - Don't over-sponsor
- **Monitor patterns** - Identify waste
- **Consider tiered sponsorship** - More for active users
