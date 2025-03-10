---
title: Run a Sui Fullnode
---

Sui full nodes run a service that
stores the full blockchain state and history. They service reads, either for
end clients or by helping other full nodes get up-to-date with the latest
transactions that have been committed to the chain.

This role enables
[validators](../learn/architecture/validators.md) (or miners in
other networks) to focus on servicing the write path and processing
transactions as fast as possible. Once a validator has committed a new set of
transactions (or a block of transactions), the validator will push that block
to a full node (potentially a number of full nodes) who will then in turn
disseminate it to the rest of the network.

## Features

Sui full nodes exist to:

* Track and verify the state of the blockchain, independently and locally.
* Serve read requests from clients.
* Conduct local app testing against verified data.

## State synchronization

Today Sui full nodes sync with validators to be able to learn about newly
committed transactions.

The normal life of a transaction requires a few round trips to 2f+1 validators
to be able to form a TxCert, at which point a transaction is guaranteed to be
committed and executed.

Today, this synchronization process is performed by:

1. Following 2f+1 validators and listening for newly committed transactions.
1. Requesting the transaction from one validator.
1. Locally executing the transaction and updating the local DB.

This synchronization process is far from ideal as it requires listening
to at a minimum 2f+1 validators to ensure that a full node has properly seen all
new transactions. Over time, we will improve this process (e.g. with the
introduction of checkpoints, ability to synchronize with other full nodes,
etc.) in order to have better guarantees around a full node’s ability to be
confident it has seen all recent transactions.

## Architecture

The Sui full node is essentially a read-only view of the network state. Unlike
validator nodes, full nodes cannot sign transactions, although they can validate
the integrity of the chain by re-executing transactions that were previously
committed by a quorum of validators.

Today, a full node is expected to maintain the full history of the chain; in the
future, sufficiently old history may need to be pruned and offloaded to cheaper
storage.

Conversely, a validator needs to store only the latest transactions on the
*frontier* of the object graph (e.g., txes with >0 unspent output objects).

## Full node setup

Follow the instructions here to run your own Sui full node.

### Hardware requirements

We recommend the following minimum hardware requirements for running a full node:

* CPUs: 2
* RAM: 8GB
* Storage: 50GB

### Software requirements

We recommend running Sui full nodes on Linux. The Sui team supports the Ubuntu and
Debian distributions and tests against
[Ubuntu version 18.04 (Bionic Beaver)](https://releases.ubuntu.com/18.04/).

That said, you are welcome to run a Sui full node on the operating system of your
choosing and submit changes to accommodate that environment. See [Install Sui](../build/install.md)
for setup instructions for each operating system.

Specifically, ensure the required tools are installed and updated in your environment as
outlined in the [Prerequisites](../build/install.md#prerequisites) section. In particular,
ensure [Rust](../build/install.md#rust) is up-to-date.

Similarly, if you are using Windows Subsystem for Linux (WSL), install a fresh copy of
[CLang/LLVM](https://releases.llvm.org/download.html), as described in [Prerequisites](../build/install.md#prerequisites).

Note, you will fork the Sui repository here rather than clone it as described in
*Prerequisites*. So you can skip that step.

If you are using Linux, install these extra dependencies. For example, in Ubuntu, run:
```shell
    $ apt-get update \
    && apt-get install -y --no-install-recommends \
    tzdata \
    git \
    ca-certificates \
    curl \
    build-essential \
    libssl-dev \
    pkg-config \
    libclang-dev \
    cmake
```

If you are using macOS or Windows Subsystem for Linux (WSL), the command will be similar. Remember to install [CLang](https://clang.llvm.org/) in WSL.

## Configure a full node

You may run a full node either by employing Docker or by building from
source.

### Using Docker Compose

Follow the instructions in the
[Full node Docker README](https://github.com/MystenLabs/sui/tree/main/docker/fullnode#readme)
to run a Sui full node using Docker, including [resetting the environment](https://github.com/MystenLabs/sui/tree/main/docker/fullnode#reset-the-environment).

### Building from source

1. Install the required tools in your environment as
outlined in the [Prerequisites](../build/install.md#prerequisites) section if you
haven't already. Make sure your entire toolchain stays up-to-date. If you encounter
issues building and installing the Sui binaries, update all packages above and re-install.
1. Set up your fork of the Sui repository:
    1. Go to the [Sui repository](https://github.com/MystenLabs/sui) on GitHub
       and click the *Fork* button in the top right-hand corner of the screen.
    1. Clone your personal fork of the Sui repository to your local machine
       (ensure that you insert your GitHub username into the URL):
       ```shell
       $ git clone https://github.com/<YOUR-GITHUB-USERNAME>/sui.git
       ```
1. `cd` into your `sui` repository:
    ```shell
    $ cd sui
    ```
1. Set up the Sui repository as a git remote:
    ```shell
    $ git remote add upstream https://github.com/MystenLabs/sui
    ```
1. Sync your fork:
    ```shell
    $ git fetch upstream
    ```
1. Check out the `devnet` branch:
    ```shell
    $ git checkout --track upstream/devnet
    ```
1. Make a copy of the [full node configuration template](https://github.com/MystenLabs/sui/blob/main/crates/sui-config/data/fullnode-template.yaml):
   ```shell
   $ cp crates/sui-config/data/fullnode-template.yaml fullnode.yaml
   ```
1. Download the latest
   [`genesis`](https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob)
   state for Devnet by clicking that link or by running the following in your
   terminal:
    ```shell
    $ curl -fLJO https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob
    ```
1. Optional: You can skip this set of steps if you are willing to accept the default paths to
    resources. If you need custom paths, edit your `fullnode.yaml` file to reflect the paths
    you employ:
    1. Update the `db-path` field with the path to where the full node's database
       is located. By default this will create the database in a directory
       `./suidb` relative to your current directory:
       ```yaml
       db-path: "/path/to/suidb"
       ```
    1. Update the `genesis-file-location` with the path to the `genesis` file.
       By default, the config looks for the file `genesis.blob` in your
       current directory:
       ```yaml
       genesis:
       genesis-file-location: "/path/to/genesis.blob"
       ```
1. Start your Sui full node:
    ```shell
    $ cargo run --release --bin sui-node -- --config-path fullnode.yaml
    ```
1. Post build, receive the success confirmation message, `SuiNode started!`
1. Optional: [Publish / subscribe](pubsub.md) to notifications using JSON-RPC via websocket.

Your full node will now be serving the read endpoints of the [Sui JSON-RPC
API](../build/json-rpc.md#sui-json-rpc-api) at:
`http://127.0.0.1:9000`

## Using Sui Explorer with your full node

[Sui Explorer](https://explorer.devnet.sui.io/) lets you configure where
it should issue read requests to query the blockchain. This enables you to
point the Explorer at your locally running full node and see the
transactions it has synced from the network. To make this change:

1. Open a browser and go to: https://explorer.devnet.sui.io/
1. Click the **Devnet** button in the top right-hand corner of Sui Explorer and select
   the *Local* network from the drop-down menu.
1. Close the *Choose a Network* menu to see the latest transactions. 

Sui Explorer now uses your local full node to explore the state of the chain.

## Monitoring

Monitor your full node using the instructions at [Logging, Tracing, Metrics, and
Observability](https://docs.sui.io/contribute/observability).

Note the default metrics port is 9184 yet configurable in your `fullnode.yaml` file.

## Updating your full node with new releases

Whenever Sui releases a new version, Devnet restarts as a new network with no data. You must update your full node with each Sui release to ensure compatibility with the network.

### With Docker Compose

Follow the instructions to [reset the environment](https://github.com/MystenLabs/sui/tree/main/docker/fullnode#reset-the-environment),
namely by running the command:
```shell
$ docker-compose down --volumes
```

### Built from source

If you followed the instructions for [Building from
Source](#building-from-source), update your full node as follows:

1. Shut down your currently running full node.
1. `cd` into your local Sui repository:
    ```shell
    $ cd sui
    ```
1. Remove the old on-disk database and 'genesis.blob' file:
    ```shell
    $ rm -r suidb genesis.blob
    ```
1. Fetch the source from the latest release:
    ```shell
    $ git fetch upstream
    ```
1. Reset your branch:
    ```shell
    $ git checkout -B devnet --track upstream/devnet
    ```
1. Download the latest
   [`genesis`](https://github.com/MystenLabs/sui-genesis/raw/main/devnet/genesis.blob)
   state for Devnet as described above.
1. Update your `fullnode.yaml` configuration file if needed.
1. Restart your Sui full node:
    ```shell
    $ cargo run --release --bin sui-node -- --config-path fullnode.yaml
    ```
Your full node will once again be running at:
`http://127.0.0.1:9000`

## Future plans

Today, a full node relies only on synchronizing with 2f+1 validators in order to
ensure it has seen all committed transactions. In the future, we expect
full nodes to fully participate in a peer-to-peer (p2p) environment where the
load of disseminating new transactions can be shared with the whole network and
not place the burden solely on the validators. We also expect future
features, such as checkpoints, to enable improved performance of synchronizing the
state of the chain from genesis.

Please see our [privacy policy](https://sui.io/policy/) to learn how we handle
information about our nodes.
