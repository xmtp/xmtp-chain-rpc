# XMTP Chain RPC Node

> **Disclaimer**: This repository is provided as a demonstration only. No support is provided.

- [XMTP Chain RPC Node](#xmtp-chain-rpc-node)
  - [Node Role](#node-role)
  - [Architecture](#architecture)
  - [XMTP Ropsten Network Details](#xmtp-ropsten-network-details)
  - [Quick Start](#quick-start)
  - [Hardware Requirements](#hardware-requirements)
  - [Configuration Parameters Explained](#configuration-parameters-explained)
    - [Chain Configuration](#chain-configuration)
    - [Data Sources](#data-sources)
    - [Transaction Forwarding](#transaction-forwarding)
    - [Archive Mode (for Indexers)](#archive-mode-for-indexers)
    - [Disabled Features (Read-Only Node)](#disabled-features-read-only-node)
    - [RPC Exposure](#rpc-exposure)
    - [Other](#other)
  - [Snapshot Initialization (Optional)](#snapshot-initialization-optional)
  - [Metrics (Optional)](#metrics-optional)
  - [Resources](#resources)
  - [Support](#support)

This repository demonstrates how to run an **XMTP Chain full node** for read-only purposes. The node syncs chain state locally and exposes JSON-RPC and WebSocket endpoints for indexers, explorers, or other services to query.

## Node Role

The configuration in this repository is designed for a **read-only full node** that:

- Syncs and stores the full chain state locally (archive mode)
- Serves JSON-RPC (HTTP) and WebSocket (WSS) requests
- Forwards any submitted transactions to the sequencer
- Does **NOT** act as a sequencer, validator, batch poster, or DAC member

This is ideal for running your own RPC endpoint for indexers, block explorers, or applications that need to read chain data without relying on third-party RPC providers.

## Architecture

XMTP Chain is built on the [Arbitrum Nitro](https://docs.arbitrum.io/) stack. Unlike traditional Ethereum P2P networks, Nitro nodes:

- Receive blocks via a **sequencer feed** (WebSocket)
- Fetch batch data from the **Data Availability** layer (REST API)
- Read parent chain state via **RPC** (Base Sepolia for XMTP Ropsten)

There is no P2P gossip or enode-based peer discovery.

## XMTP Ropsten Network Details

| Component       | Value                                                |
| --------------- | ---------------------------------------------------- |
| Chain ID        | `351243127`                                          |
| Parent Chain    | Base Sepolia (`84532`)                               |
| Sequencer Feed  | `wss://xmtp-ropsten-relay.rollups.alchemy.com`       |
| DA Mirror       | `https://xmtp-ropsten-da-mirror.rollups.alchemy.com` |
| Container Image | `offchainlabs/nitro-node:v3.9.4-7f582c3`             |

## Quick Start

> **Important**: The provided Base Sepolia RPC URL (`https://base-sepolia.drpc.org`) is included as an example only. Public RPC endpoints typically have rate limits that will cause sync issues. You should replace it with your own RPC endpoint from a provider like [Alchemy](https://www.alchemy.com/), [Infura](https://www.infura.io/), or run your own Base Sepolia node.

```bash
./start-xmtp-ropsten
```

This exposes:

- **HTTP JSON-RPC**: `http://localhost:8547`
- **WebSocket**: `ws://localhost:8548`

Data is persisted in `./data`.

## Hardware Requirements

Arbitrum recommends the following minimum specifications for running a full node:

| Resource     | Recommended                                             |
| ------------ | ------------------------------------------------------- |
| RAM          | 64 GB                                                   |
| CPU          | 8 core CPU (for AWS, an `i4i.2xlarge` instance)         |
| Storage Type | NVMe SSD (locally attached drives strongly recommended) |
| Storage Size | Depends on the chain and its traffic over time          |

> [!NOTE]
> Archive nodes require significantly more storage than pruned nodes, as they retain all historical state. While full archive nodes are recommended for complete historical access, **pruned nodes with ~6 months of data** are a valid alternative if storage is limited. See [Archive Mode](#archive-mode-for-indexers) for configuration details.

## Configuration Parameters Explained

### Chain Configuration

| Flag                            | Description                                                                       |
| ------------------------------- | --------------------------------------------------------------------------------- |
| `--chain.info-json`             | JSON containing chain config, rollup contract addresses, and genesis parameters   |
| `--chain.name`                  | Chain identifier (`xmtp-ropsten`)                                                 |
| `--parent-chain.connection.url` | RPC endpoint for the parent chain (Base Sepolia). **Use a non-rate-limited RPC.** |

### Data Sources

| Flag                                              | Description                                             |
| ------------------------------------------------- | ------------------------------------------------------- |
| `--node.feed.input.url`                           | WebSocket URL to receive new blocks from the sequencer  |
| `--node.data-availability.enable`                 | Enable Data Availability mode (required for DAC chains) |
| `--node.data-availability.rest-aggregator.enable` | Enable fetching batch data via REST                     |
| `--node.data-availability.rest-aggregator.urls`   | DA mirror endpoint(s) for batch retrieval               |

### Transaction Forwarding

| Flag                            | Description                                            |
| ------------------------------- | ------------------------------------------------------ |
| `--execution.forwarding-target` | URL to forward submitted transactions to the sequencer |

### Archive Mode (for Indexers)

| Flag                                       | Description                                             |
| ------------------------------------------ | ------------------------------------------------------- |
| `--execution.caching.archive`              | Retain historical state for queries at past blocks      |
| `--execution.tx-indexer.tx-lookup-limit=0` | Keep transaction index for all blocks (`0` = unlimited) |

> [!NOTE]
> While a full archive node is recommended, a **pruned node with ~6 months of historical data** is also a valid option if storage is a constraint. To configure a pruned node, omit `--execution.caching.archive` and set `--execution.tx-indexer.tx-lookup-limit` to a non-zero value representing the number of blocks to retain (e.g., `--execution.tx-indexer.tx-lookup-limit=62208000` for ~6 months at 0.25s block time).

### Disabled Features (Read-Only Node)

| Flag                                    | Description                              |
| --------------------------------------- | ---------------------------------------- |
| `--execution.sequencer.enable=false`    | Do not act as sequencer                  |
| `--node.staker.enable=false`            | Do not participate in validation/staking |
| `--node.batch-poster.enable=false`      | Do not post batches to L1                |
| `--node.delayed-sequencer.enable=false` | Do not process delayed inbox messages    |

### RPC Exposure

| Flag                                | Description                                 |
| ----------------------------------- | ------------------------------------------- |
| `--http.addr=0.0.0.0`               | Bind HTTP RPC to all interfaces             |
| `--http.api=net,web3,eth,debug,arb` | Enabled HTTP RPC namespaces                 |
| `--http.corsdomain=*`               | Allow CORS from any origin                  |
| `--http.vhosts=*`                   | Accept requests for any hostname            |
| `--ws.addr=0.0.0.0`                 | Bind WebSocket to all interfaces            |
| `--ws.api=net,web3,eth,debug,arb`   | Enabled WebSocket RPC namespaces            |
| `--ws.origins=*`                    | Allow WebSocket connections from any origin |

### Other

| Flag                                   | Description                                            |
| -------------------------------------- | ------------------------------------------------------ |
| `--node.dangerous.disable-blob-reader` | Disable EIP-4844 blob reading (not used on this chain) |

## Snapshot Initialization (Optional)

> Note: snapshots are not available for now, so syncing from genesis the default.

To speed up initial sync, you can provide a snapshot:

```bash
--init.url=https://example.com/xmtp-ropsten-snapshot.tar
```

Or search for the latest snapshot (if available):

```bash
--init.latest=archive
--init.latest-base=https://your-snapshot-server/
```

## Metrics (Optional)

To enable Prometheus metrics:

```bash
--metrics \
--metrics-server.addr=0.0.0.0 \
--metrics-server.port=6070
```

## Resources

- [Arbitrum Documentation](https://docs.arbitrum.io/)
- [Run a Full Node (Arbitrum Docs)](https://docs.arbitrum.io/run-arbitrum-node/run-full-node)
- [Nitro Node Docker Images](https://hub.docker.com/r/offchainlabs/nitro-node/tags)

## Support

**No support is provided for this repository.** This is a demonstration only. For questions about running Arbitrum Nitro nodes, refer to the [official Arbitrum documentation](https://docs.arbitrum.io/).
