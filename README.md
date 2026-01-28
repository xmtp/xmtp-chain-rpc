# XMTP Chain RPC Node

> **Disclaimer**: This repository is provided as a demonstration only. No support is provided.

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

| Component | Value |
| ----------- | ------- |
| Chain ID | `351243127` |
| Parent Chain | Base Sepolia (`84532`) |
| Sequencer Feed | `wss://xmtp-ropsten-relay.rollups.alchemy.com` |
| DA Mirror | `https://xmtp-ropsten-da-mirror.rollups.alchemy.com` |
| Container Image | `offchainlabs/nitro-node:v3.9.4-7f582c3` |

## Quick Start

```bash
./start-xmtp-ropsten
```

This exposes:

- **HTTP JSON-RPC**: `http://localhost:8547`
- **WebSocket**: `ws://localhost:8548`

Data is persisted in `./data`.

## Configuration Parameters Explained

### Chain Configuration

| Flag | Description |
| ------ | ------------- |
| `--chain.info-json` | JSON containing chain config, rollup contract addresses, and genesis parameters |
| `--chain.name` | Chain identifier (`xmtp-ropsten`) |
| `--parent-chain.connection.url` | RPC endpoint for the parent chain (Base Sepolia) |

### Data Sources

| Flag | Description |
| ------ | ------------- |
| `--node.feed.input.url` | WebSocket URL to receive new blocks from the sequencer |
| `--node.data-availability.enable` | Enable Data Availability mode (required for DAC chains) |
| `--node.data-availability.rest-aggregator.enable` | Enable fetching batch data via REST |
| `--node.data-availability.rest-aggregator.urls` | DA mirror endpoint(s) for batch retrieval |

### Transaction Forwarding

| Flag | Description |
| ------ | ------------- |
| `--execution.forwarding-target` | URL to forward submitted transactions to the sequencer |

### Archive Mode (for Indexers)

| Flag | Description |
| ------ | ------------- |
| `--execution.caching.archive` | Retain historical state for queries at past blocks |
| `--execution.tx-indexer.tx-lookup-limit=0` | Keep transaction index for all blocks (`0` = unlimited) |

### Disabled Features (Read-Only Node)

| Flag | Description |
| ------ | ------------- |
| `--execution.sequencer.enable=false` | Do not act as sequencer |
| `--node.staker.enable=false` | Do not participate in validation/staking |
| `--node.batch-poster.enable=false` | Do not post batches to L1 |
| `--node.delayed-sequencer.enable=false` | Do not process delayed inbox messages |

### RPC Exposure

| Flag | Description |
| ------ | ------------- |
| `--http.addr=0.0.0.0` | Bind HTTP RPC to all interfaces |
| `--http.api=net,web3,eth,debug,arb` | Enabled HTTP RPC namespaces |
| `--http.corsdomain=*` | Allow CORS from any origin |
| `--http.vhosts=*` | Accept requests for any hostname |
| `--ws.addr=0.0.0.0` | Bind WebSocket to all interfaces |
| `--ws.api=net,web3,eth,debug,arb` | Enabled WebSocket RPC namespaces |
| `--ws.origins=*` | Allow WebSocket connections from any origin |

### Other

| Flag | Description |
| ------ | ------------- |
| `--node.dangerous.disable-blob-reader` | Disable EIP-4844 blob reading (not used on this chain) |

## Snapshot Initialization (Optional)

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
