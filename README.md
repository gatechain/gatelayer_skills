# GateLayer Skills


[Agent Skills](https://agentskills.io) for building on [GateLayer](https://www.gatechain.io/docs/GateLayer/Introduction/). These skills enable AI agents to connect to GateLayer, deploy contracts, and interact with the network.

## Available Skills

| Skill | Description |
| ----- | ----------- |
| [Connecting to GateLayer Network](./skills/connecting-to-gatelayer-network/SKILL.md) | Provides GateLayer Mainnet and Testnet network configuration, RPC endpoints, chain IDs, and explorer URLs. |
| [Deploying Contracts on GateLayer](./skills/deploying-contracts-on-gatelayer/SKILL.md) | Deploys  contracts on GateLayer with Foundry, plus common troubleshooting guidance. |
| [Running a Gatelayer Node](./skills/running-a-gatelayer-node/SKILL.md) | Covers production node setup, hardware requirements, networking ports, and syncing guidance. |

## Installation

Install with [Vercel's Skills CLI](https://skills.sh):

```bash
npx skills add gatechain/gatelayer_skills
```

## Usage

Skills are automatically available once installed. The agent will use them when relevant tasks are detected.

**Examples:**

```text
Deploy my contract to GateLayer testnet
```

```text
How do I connect to GateLayer mainnet?
```



## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

## License

This project is licensed under the terms of the included LICENSE file.

---
[GateLayer]:https://www.gatechain.io/docs/GateLayer/Introduction/ 
