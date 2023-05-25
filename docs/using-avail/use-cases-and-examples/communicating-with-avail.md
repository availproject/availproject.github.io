---
id: communicating-with-avail
title: Communicating With Avail Examples
sidebar_label: Communicating With Avail
description: Examples of how to communicate with Avail
keywords:

- docs
- avail
- examples
image: https://availproject.github.io/img/avail/AvailDocs.png
slug: /using-avail/communicating-with-avail

---

# Communicating With Avail

The API provides application developers the ability to query a node and interact with the Avail.
Here you will find a list of examples in `Javascript/Typesctipt`, `Rust` and `Go` that show
the basics of connecting to a local node, retrieving data from the node/chain and executing transactions on the chain.

## Prerequisites

For the following examples, you need to run a local node or update a configuration to match the running network.
Detail instructions on how to run examples can be found in the Avail repository
for [Typescript](https://github.com/availproject/avail/blob/develop/examples/ts/README.md)
for [Go](https://github.com/availproject/avail/blob/develop/examples/go/README.md)
for [Rust](https://github.com/availproject/avail/blob/develop/avail-subxt/examples/README.md).

## Examples

Examples of Javascript/Typescript can be found in the `Avail`
github [repository](https://github.com/availproject/avail/tree/develop/examples/ts)
and, there are also examples written in [Go](https://github.com/availproject/avail/tree/develop/examples/go)
and [Rust](https://github.com/availproject/avail/tree/develop/avail-subxt/examples).

Some of the examples that show communication with Avail by submitting transactions and rpc calls:

- Submitting a blob of data
  transaction [example](https://github.com/availproject/avail/tree/develop/examples/data_submit.ts).
- Dispatching data root
  transaction [example](https://github.com/availproject/avail/tree/develop/examples/dispatch_data_root.ts).
- Creating application key transaction [example](https://github.com/availproject/avail/tree/develop/examples/app_id.ts).
- Creating a simple transfer [example](https://github.com/availproject/avail/tree/develop/examples/ts/src/transfer.ts).
- Query data proof [example](https://github.com/availproject/avail/tree/develop/examples/ts/src/query_proof_data.ts).
- Query proof [example](https://github.com/availproject/avail/tree/develop/examples/ts/src/query_proof.ts).
- Query app data [example](https://github.com/availproject/avail/tree/develop/examples/ts/src/query_app_data.ts).
- Listening to a new
  blocks [example](https://github.com/availproject/avail/tree/develop/examples/ts/src/listen_new_blocks.ts).
- Getting chain and node
  information [example](https://github.com/availproject/avail/tree/develop/examples/ts/src/connect.ts).
