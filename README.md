# Nevocoin

Copyright (c) 2023, The Nevocoin developers
Copyright (c) 2014-2023, The Monero Project
Portions Copyright (c) 2012-2013 The Cryptonote developers.

## Table of Contents

  - [Download](#download)
  - [Introduction](#introduction)
  - [About this project](#about-this-project)
  - [License](#license)
  - [Building](#building)
  - [Running nevocoind](#running-nevocoind)
  - [Internationalization](#internationalization)
  - [Using Tor](#using-tor)
  - [Pruning](#pruning)
  - [Debugging](#debugging)

## Download

https://github.com/nevocoin/nevocoin/releases

## Introduction

Nevocoin is a private, secure, untraceable, decentralised digital currency. You are your bank, you control your funds, and nobody can trace your transfers unless you allow them to do so.

**Privacy:** Nevocoin uses a cryptographically sound system to allow you to send and receive funds without your transactions being easily revealed on the blockchain (the ledger of transactions that everyone has). This ensures that your purchases, receipts, and all transfers remain private by default.

**Security:** Using the power of a distributed peer-to-peer consensus network, every transaction on the network is cryptographically secured. Individual wallets have a 25-word mnemonic seed that is only displayed once and can be written down to backup the wallet. Wallet files should be encrypted with a strong passphrase to ensure they are useless if ever stolen.

**Untraceability:** By taking advantage of ring signatures, a special property of a certain type of cryptography, Nevocoin is able to ensure that transactions are not only untraceable but have an optional measure of ambiguity that ensures that transactions cannot easily be tied back to an individual user or computer.

**Decentralization:** The utility of Nevocoin depends on its decentralised peer-to-peer consensus network - anyone should be able to run the nevocoin software, validate the integrity of the blockchain, and participate in all aspects of the nevocoin network using consumer-grade commodity hardware. Decentralization of the nevocoin network is maintained by software development that minimizes the costs of running the nevocoin software and inhibits the proliferation of specialized, non-commodity hardware.

## About this project

This is the core implementation of Nevocoin. It is open source and completely free to use without restrictions, except for those specified in the license agreement below. There are no restrictions on anyone creating an alternative implementation of Nevocoin that uses the protocol and network in a compatible manner.

As with many development projects, the repository on GitHub is considered to be the "staging" area for the latest changes. Before changes are merged into that branch on the main repository, they are tested by individual developers in their own branches, submitted as a pull request, and then subsequently tested by contributors who focus on testing and code reviews. That having been said, the repository should be carefully considered before using it in a production environment, unless there is a patch in the repository for a particular show-stopping issue you are experiencing. It is generally a better idea to use a tagged release for stability.

**Anyone is welcome to contribute to Nevocoin's codebase!** If you have a fix or code change, feel free to submit it as a pull request directly to the "master" branch. In cases where the change is relatively small or does not affect other parts of the codebase, it may be merged in immediately by any one of the collaborators. On the other hand, if the change is particularly large or complex, it is expected that it will be discussed at length either well in advance of the pull request being submitted, or even directly on the pull request.

## License

See [LICENSE](LICENSE).

## Building
Building Nevocoin on Ubuntu/Debian:
```
sudo apt update && sudo apt install build-essential cmake pkg-config libssl-dev libzmq3-dev libunbound-dev libsodium-dev libunwind8-dev liblzma-dev libreadline6-dev libexpat1-dev libpgm-dev qttools5-dev-tools libhidapi-dev libusb-1.0-0-dev libprotobuf-dev protobuf-compiler libudev-dev libboost-chrono-dev libboost-date-time-dev libboost-filesystem-dev libboost-locale-dev libboost-program-options-dev libboost-regex-dev libboost-serialization-dev libboost-system-dev libboost-thread-dev python3 ccache doxygen graphviz

git clone --recursive https://github.com/nevocoin/nevocoin

cd nevocoin

make
```

### With Gitian
See [contrib/gitian/README.md](contrib/gitian/README.md).

## Running nevocoind

The build places the binary in `bin/` sub-directory within the build directory
from which cmake was invoked (repository root by default). To run in the
foreground:

```bash
./bin/nevocoind
```

To list all available options, run `./bin/nevocoind --help`.  Options can be
specified either on the command line or in a configuration file passed by the
`--config-file` argument.  To specify an option in the configuration file, add
a line with the syntax `argumentname=value`, where `argumentname` is the name
of the argument without the leading dashes, for example, `log-level=1`.

To run in background:

```bash
./bin/nevocoind --log-file nevocoind.log --detach
```

If you're on Mac, you may need to add the `--max-concurrency 1` option to
nevocoin-wallet-cli, and possibly nevocoind, if you get crashes refreshing.

## Internationalization

See [README.i18n.md](docs/README.i18n.md).

## Using Tor

> There is a new, still experimental, [integration with Tor](docs/ANONYMITY_NETWORKS.md). The
> feature allows connecting over IPv4 and Tor simultaneously - IPv4 is used for
> relaying blocks and relaying transactions received by peers whereas Tor is
> used solely for relaying transactions received over local RPC. This provides
> privacy and better protection against surrounding node (sybil) attacks.

While Nevocoin isn't made to integrate with Tor, it can be used wrapped with torsocks, by
setting the following configuration parameters and environment variables:

* `--p2p-bind-ip 127.0.0.1` on the command line or `p2p-bind-ip=127.0.0.1` in
  nevocoind.conf to disable listening for connections on external interfaces.
* `--no-igd` on the command line or `no-igd=1` in nevocoind.conf to disable IGD
  (UPnP port forwarding negotiation), which is pointless with Tor.
* `DNS_PUBLIC=tcp` or `DNS_PUBLIC=tcp://x.x.x.x` where x.x.x.x is the IP of the
  desired DNS server, for DNS requests to go over TCP, so that they are routed
  through Tor. When IP is not specified, nevocoind uses the default list of
  servers defined in [src/common/dns_utils.cpp](src/common/dns_utils.cpp).
* `TORSOCKS_ALLOW_INBOUND=1` to tell torsocks to allow nevocoind to bind to interfaces
   to accept connections from the wallet. On some Linux systems, torsocks
   allows binding to localhost by default, so setting this variable is only
   necessary to allow binding to local LAN/VPN interfaces to allow wallets to
   connect from remote hosts. On other systems, it may be needed for local wallets
   as well.
* Do NOT pass `--detach` when running through torsocks with systemd, (see
  [utils/systemd/nevocoind.service](utils/systemd/nevocoind.service) for details).
* If you use the wallet with a Tor daemon via the loopback IP (eg, 127.0.0.1:9050),
  then use `--untrusted-daemon` unless it is your own hidden service.

Example command line to start nevocoind through Tor:

```bash
DNS_PUBLIC=tcp torsocks nevocoind --p2p-bind-ip 127.0.0.1 --no-igd
```

A helper script is in contrib/tor/nevocoin-over-tor.sh. It assumes Tor is installed
already, and runs Tor and Nevocoin with the right configuration.

### Using Tor on Tails

TAILS ships with a very restrictive set of firewall rules. Therefore, you need
to add a rule to allow this connection too, in addition to telling torsocks to
allow inbound connections. Full example:

```bash
sudo iptables -I OUTPUT 2 -p tcp -d 127.0.0.1 -m tcp --dport 18081 -j ACCEPT
DNS_PUBLIC=tcp torsocks ./nevocoind --p2p-bind-ip 127.0.0.1 --no-igd --rpc-bind-ip 127.0.0.1 \
    --data-dir /home/amnesia/Persistent/your/directory/to/the/blockchain
```

## Pruning

To save disk space, one can store a pruned Nevocoin blockchain.
You can start a pruned node with the following command:
```bash
nevocoind --sync-pruned-blocks --prune-blockchain
```
Please note that once the blockchain is pruned, it cannot be restored without re-syncing.

## Debugging

This section contains general instructions for debugging failed installs or problems encountered with Nevocoin. First, ensure you are running the latest version built from the GitHub repo.

### Obtaining stack traces and core dumps on Unix systems

We generally use the tool `gdb` (GNU debugger) to provide stack trace functionality, and `ulimit` to provide core dumps in builds which crash or segfault.

* To use `gdb` in order to obtain a stack trace for a build that has stalled:

Run the build.

Once it stalls, enter the following command:

```bash
gdb /path/to/nevocoind `pidof nevocoind`
```

Type `thread apply all bt` within gdb in order to obtain the stack trace

* If however the core dumps or segfaults:

Enter `ulimit -c unlimited` on the command line to enable unlimited filesizes for core dumps

Enter `echo core | sudo tee /proc/sys/kernel/core_pattern` to stop cores from being hijacked by other tools

Run the build.

When it terminates with an output along the lines of "Segmentation fault (core dumped)", there should be a core dump file in the same directory as nevocoind. It may be named just `core`, or `core.xxxx` with numbers appended.

You can now analyse this core dump with `gdb` as follows:

```bash
gdb /path/to/nevocoind /path/to/dumpfile`
```

Print the stack trace with `bt`

 * If a program crashed and cores are managed by systemd, the following can also get a stack trace for that crash:

```bash
coredumpctl -1 gdb
```

#### To run Nevocoin within gdb:

Type `gdb /path/to/nevocoind`

Pass command-line options with `--args` followed by the relevant arguments

Type `run` to run nevocoind

### Analysing memory corruption

There are two tools available:

#### ASAN

Configure Nevocoin with the -D SANITIZE=ON cmake flag, eg:

```bash
cd build/debug && cmake -D SANITIZE=ON -D CMAKE_BUILD_TYPE=Debug ../..
```

You can then run the nevocoin tools normally. Performance will typically halve.

#### valgrind

Install valgrind and run as `valgrind /path/to/nevocoind`. It will be very slow.

### LMDB

Instructions for debugging suspected blockchain corruption as per @HYC

There is an `mdb_stat` command in the LMDB source that can print statistics about the database but it's not routinely built. This can be built with the following command:

```bash
cd ~/nevocoin/external/db_drivers/liblmdb && make
```

The output of `mdb_stat -ea <path to blockchain dir>` will indicate inconsistencies in the blocks, block_heights and block_info table.

The output of `mdb_dump -s blocks <path to blockchain dir>` and `mdb_dump -s block_info <path to blockchain dir>` is useful for indicating whether blocks and block_info contain the same keys.

These records are dumped as hex data, where the first line is the key and the second line is the data.
