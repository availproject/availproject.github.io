---
id: avail-archive-node
title: Run a Archive Node
sidebar_label: Run a Archive Node
description: Learn how to run a archive node
keywords:
  - docs
  - avail
  - node
  - data availability
  - da
image: https://availproject.github.io/img/avail/AvailDocs.png
slug: avail-archive-node
---
import useBaseUrl from '@docusaurus/useBaseUrl';

## Archive Node Setup

To run an archive node you must first reveiw the [full node](/join-the-network/run-avail/full-node-setup) setup. You will complete all the same steps. The only difference is you will add `--pruning archive` to the paramaters. 

When running as a full node, only the state of the past 256 blocks will be kept. Adding `--pruning archive` will store all the data.

Example for an archive node start command:
```
./data-avail --base-path ~/avail-node/data \
                --chain ~/avail-node/chainspec.raw.json \
                --pruning archive \
                --port 30333 \
                --bootnodes /dns/gateway-fullnode-002.testnet.avail.tools/tcp/30333/p2p/12D3KooWNuBaLtAGNxQbei7rUzpp8N8TF8k5kPsgKShAJgK4crkB \
                /dns/gateway-fullnode-001.testnet.avail.tools/tcp/30333/p2p/12D3KooWDgqCRtsJWKjckh2XHtRZbboVdgDJswsxoNmX8PMf59bV \
                /dns/gateway-fullnode-003.testnet.avail.tools/tcp/30333/p2p/12D3KooWBNy1vzragtwiummqXwry19h6dke68hybY6jVeEH4mAtT
```

:::info Storage space requirements

An archive node will require more storage over time and the storage space requirements will grow over time.
:::
