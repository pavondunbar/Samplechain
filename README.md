# Samplechain

Samplechain is a Substrate-based, Ethereum-compatible Layer 1 blockchain. It combines the Polkadot SDK with the Frontier EVM stack to deliver a chain that natively supports both Substrate pallets and Solidity smart contracts.

## Key Features

- **Dual Execution Environment** -- Run native Substrate pallets alongside Solidity/EVM smart contracts on the same chain. Developers can use whichever environment fits their use case.
- **BABE + GRANDPA Consensus** -- Hybrid consensus using BABE for probabilistic block production (6-second block time) and GRANDPA for deterministic finality.
- **Full Ethereum JSON-RPC** -- Connect MetaMask, Hardhat, Foundry, and other Ethereum tooling directly via standard `eth_*` RPC methods. EVM Chain ID: `1298`.
- **On-Chain Governance** -- Democratic referenda, council voting, technical committee, treasury management, and conviction-based voting built into the runtime.
- **Nominated Proof-of-Stake** -- Up to 1,000 active validators with nomination pools, slashing, bonding, and era-based reward distribution.
- **Halving Reward Schedule** -- Initial reward of ~1,370 SMPL per era, halving every 17,520 eras. Validator share: 45%, Proof of Value share: 55%.
- **Custom Pallet (pallet-counter)** -- Token minting/burning, balance locking, cross-chain Substrate-to-EVM transfers, IPFS hash verification, and content-filtered message passing.
- **NFT and Asset Support** -- Built-in pallets for fungible assets, NFTs, NFT fractionalization, and on-chain asset conversion/swaps.
- **Identity and Recovery** -- On-chain identity registration with proxy accounts, multisig wallets, and social recovery.

## Token

| Property | Value |
|----------|-------|
| Name | SampleCoin |
| Symbol | SMPL |
| Decimals | 18 |
| SS58 Format | 33 |

## Network Parameters

| Parameter | Value |
|-----------|-------|
| Block Time | 6 seconds |
| Epoch Duration | 100 blocks (~10 minutes) |
| Sessions Per Era | 6 |
| Bonding Duration | 672 eras (~28 days) |
| Slash Defer Duration | 168 eras (~7 days) |
| Max Active Validators | 1,000 |
| Max Nominators per Validator | 64 |
| EVM Chain ID | 1298 |
| Block Gas Limit | 75,000,000 |
| Base Fee | 1 Gwei |

## Hardware Requirements

| Component | Minimum |
|-----------|---------|
| CPU | 16 cores |
| RAM | 32 GB |
| Storage | 1 TB SSD |
| Network | 1 Gbps |

## Prerequisites

- **Rust 1.85.0** (managed via `rust-toolchain.toml` in the repo)
- **WASM target**: `wasm32-unknown-unknown` (included in the toolchain config)
- **System dependencies** (Ubuntu/Debian):
  ```bash
  sudo apt update && sudo apt install -y \
    build-essential git clang curl libssl-dev \
    llvm libudev-dev protobuf-compiler pkg-config
  ```
- **System dependencies** (macOS):
  ```bash
  brew install openssl protobuf
  ```

## Build from Source

Clone the repository and build the node binary:

```bash
git clone https://github.com/Devolved-AI/Samplechain.git
cd Samplechain
cargo build --release
```

The compiled binary will be at `target/release/samplechain`.

> Build times depend on hardware. First builds compile the entire Polkadot SDK and Frontier stack. Subsequent builds are incremental.

## Running a Node

### Join the Live Network

```bash
./target/release/samplechain \
  --chain samplechain \
  --name "MyNode" \
  --telemetry-url "wss://telemetry.polkadot.io/submit/ 0"
```

### Run a Local Development Node

```bash
./target/release/samplechain --dev
```

This starts a single-validator development chain with pre-funded accounts (Alice, Bob, Charlie, Dave, Eve, Ferdie) each holding 1,000,000 SMPL.

### Common CLI Flags

| Flag | Description |
|------|-------------|
| `--validator` | Enable validator mode |
| `--rpc-port 9944` | Set WebSocket RPC port |
| `--rpc-cors all` | Allow all CORS origins for RPC |
| `--pruning archive` | Keep full block history |
| `--name "MyNode"` | Set a human-readable node name |
| `--base-path /data/samplechain` | Set the data directory |

## Running a Validator

1. **Sync a full node** first and wait for it to catch up with the chain head.

2. **Generate session keys** by calling the RPC method:
   ```bash
   curl -H "Content-Type: application/json" \
     -d '{"id":1, "jsonrpc":"2.0", "method":"author_rotateKeys"}' \
     http://localhost:9944
   ```

3. **Bond SMPL tokens** -- Minimum stash of 50,000 SMPL required for validators on the staging network.

4. **Set session keys** on-chain using the `session.setKeys` extrinsic.

5. **Signal intent to validate** using the `staking.validate` extrinsic.

For a detailed walkthrough, see the [Samplechain Validator Guide](https://devolved-ai.gitbook.io/samplechain-validator-guide).

## EVM / Ethereum Compatibility

Samplechain runs a full EVM accessible via standard Ethereum JSON-RPC. Connect any Ethereum wallet or development tool using:

| Setting | Value |
|---------|-------|
| RPC URL | `http://<node-ip>:9944` |
| Chain ID | `1298` |
| Currency Symbol | `SMPL` |

### Deploying Solidity Contracts

Use Hardhat, Foundry, or Remix with the RPC URL pointed at your Samplechain node. All standard EVM opcodes and precompiles are supported:

- ECRecover, SHA256, RIPEMD160, Identity, Modexp
- SHA3-FIPS, ECRecoverPublicKey
- BN128, Blake2, BLS12-377, BLS12-381, BW6-761, Ed25519, Curve25519

## Runtime Pallets

Samplechain's runtime includes 50+ pallets:

**Core**: System, Timestamp, Balances, Transaction Payment, Indices, Utility, Multisig, Proxy, Scheduler

**Consensus**: BABE, GRANDPA, Session, Authority Discovery, Im Online, Offences, Historical

**Staking**: Staking, Election Provider Multi-Phase, Bags List, Nomination Pools, Fast Unstake, Delegated Staking

**Governance**: Democracy, Collective (Council + Technical Committee), Elections Phragmen, Treasury, Tips, Bounties, Child Bounties, Conviction Voting, Referenda, Ranked Collective, Whitelist, Alliance

**Assets**: Assets, Uniques, NFTs, NFT Fractionalization, Asset Conversion, Asset Rate

**Identity & Social**: Identity, Recovery, Society, Membership, Vesting

**EVM (Frontier)**: Ethereum, EVM, EVM Chain ID, Base Fee, Dynamic Fee

**Custom**: Pallet Counter (mint/burn, lock/unlock, cross-chain transfers, IPFS verification)

**Safety**: Safe Mode, TX Pause, Sudo, Root Testing

## Project Structure

```
Samplechain/
├── substrate/bin/node/
│   ├── cli/              # Node binary, CLI, chain specs
│   ├── runtime/          # Runtime logic (pallets, weights, constants)
│   ├── rpc/              # JSON-RPC endpoint definitions
│   ├── primitives/       # Core types (AccountId, Balance, etc.)
│   └── testing/          # Integration test utilities
├── frame/                # Frontier + custom FRAME pallets
│   ├── ethereum/         # Ethereum transaction pallet
│   ├── evm/              # EVM execution + precompiles
│   ├── base-fee/         # EIP-1559 base fee
│   ├── dynamic-fee/      # Dynamic fee adjustment
│   └── pallet-counter/   # Custom Samplechain pallet
├── client/               # Frontier client components
│   ├── rpc/              # Ethereum RPC implementation
│   ├── db/               # SQL-backed EVM state database
│   ├── mapping-sync/     # EVM block/state mapping
│   └── consensus/        # EVM consensus integration
├── primitives/           # Frontier primitive types
├── precompiles/          # EVM precompile implementations
└── substrate/            # Vendored Polkadot SDK
```

## Benchmarking

Run runtime benchmarks to generate accurate weight values:

```bash
./target/release/samplechain benchmark pallet \
  --chain dev \
  --pallet "*" \
  --extrinsic "*" \
  --steps 50 \
  --repeat 20 \
  --output weights/
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines. All contributions are licensed under Apache 2.0 and must be compatible with GPLv3.

## License

- **Runtime**: Apache-2.0
- **Node CLI**: GPL-3.0-or-later WITH Classpath-exception-2.0
- **Substrate/Polkadot SDK**: GPL-3.0-only
