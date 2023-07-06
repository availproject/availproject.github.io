---
id: avail-bootstrap-node
title: Run a Bootstrap Node
sidebar_label: Run a Bootstrap Node
description: Learn how to run a boostrap node
keywords:
  - docs
  - avail
  - node
  - data availability
  - da
image: https://availproject.github.io/img/avail/AvailDocs.png
slug: avail-bootstrap-node
---
import useBaseUrl from '@docusaurus/useBaseUrl';

## Bootstrap Node Setup

If for some reason you ever wanted to host your own network, you will need a node that will serve as an access point to all the other nodes wanting to join.
This is where the Bootstrap node comes onto to the stage. We offer a simple and functioning setup for a such node, just be sure to deploy few of these and that they are publicly accessible over the network.

First, binary for the Bootstrap node needs to be compiled. On your local machine, or either on any cloud provider VM, be sure that Rust is installed:
```
curl https://sh.rustup.rs -sSf | sh
source $HOME/.cargo/env
rustup update nightly
rustup target add wasm32-unknown-unknown --toolchain nightly
rustc --version # verify rust is working, print the installed version
```

Then download the project folder from [GitHub](https://github.com/availproject/avail-light-bootstrap), or clone the git repo:
```
git clone https://github.com/availproject/avail-light-bootstrap.git
```

Once that is done, change into downloaded project and compile: 
```
cd avail-light-bootstrap
## Inside the rust avail-light-bootstrap
cargo build --release
```
After a succesful compilation, you should be able to find the binary at `target/release/avail-light-bootstrap`. In case you buil it localy, copy the binary file to your server.
On your server, under the same folder where you have copied Bootstrap binary, create and edit a `config.yaml` file:
```
touch config.yaml
nano config.yaml

```

You can use our example file:
```yaml
log_level = 'INFO'
log_format_json = false
secret_key = { seed = "0" }
libp2p_port = 39000
libp2p_identify_protocol = '/avail_kad/id/1.0.0'
```
or if you want to change something, please be sure to the check out the [config reference](https://github.com/availproject/avail-light-bootstrap#config-reference).
And that would be it. 🎉
Happy Bootstrapping! 🚀