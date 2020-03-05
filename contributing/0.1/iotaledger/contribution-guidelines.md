# Contributing to IOTA

Thanks for taking the time to contribute! :heart:

The following set of guidelines will help you to contribute to the repositories in the [IOTAledger organization](https://github.com/iotaledger) on GitHub.

<details>
<summary>About the IOTAledger organization</summary>
<br>

IOTA is a large open-source project that's made up of over [100 repositories](https://github.com/iotaledger).

To help you find a repository to contribute to, this section organizes some of the most important ones into categories.

### Node software

Node software gives devices read/write access to the Tangle, allows them to validate transactions, and allows them to store transactions in their local ledgers.

- [**IRI:**](https://github.com/iotaledger/iri) Java software that runs on the IOTA Mainnet and Devnet. This software defines the current IOTA protocol, which allows clients to transfer the IOTA token among each other.

https://github.com/iotaledger/compass

- [**GoShimmer:**](https://github.com/iotaledger/goshimmer) Go software that implements a prototype of an IOTA network without the Coordinator. This network is still in development, so not all functionality is ready.

- [**Chronicle:**](https://github.com/iotaledger/chronicle) Elixir software that allows you to take all transactions from an IRI node and store them in a separate distributed database, which is secure and scalable

- [**Hive.go:**](https://github.com/iotaledger/hive.go) Go software that contains features, which are shared by GoShimmer and the community-built [Hornet](https://github.com/gohornet/hornet) node software

- [**Bee:**](https://github.com/iotaledger/bee) Rust software that is being developed as a successor to IRI

### Simulators

Simulators are programs that the Research Department use to test Coordicide modules.

- [**Autopeering:**](https://github.com/iotaledger/autopeering-sim) Allows you to test autopeering by simulating many nodes connecting to each other

- [**Fast Probabilistic Consensus:**](https://github.com/iotaledger/fpc-sim) Allows you to test the FPC protocol by simulating attacks

### Client libraries

Client libraries make it easy for you to integrate IOTA into your own applications. You can use them to create, send, receive, and request transactions.

- [**C**](https://github.com/iotaledger/iota.c)

- [**Go**](https://github.com/iotaledger/iota.go)

- [**Java**](https://github.com/iotaledger/iota-java)

- [**JavaScript**](https://github.com/iotaledger/iota.js)

- [**Python**](https://github.com/iotaledger/iota.py)

- [**Rust**](https://github.com/iotaledger/iota.rs)

### Wallets

A wallet is an application that stores your seed and keeps a record of transactions in a database. You can use wallets to simplify the process of securing your seed, sending transactions, and managing your balance.

- [**Hub:**](https://github.com/iotaledger/hub) A wallet management system for cryptocurrency exchanges and custodians, giving you a secure way to manage deposits and withdrawals of users' IOTA tokens

- [**Spark:**](https://github.com/iotaledger/spark-wallet) A low-security wallet intended for short-term use and sending small amounts of IOTA tokens

- [**Trinity wallet:**](https://github.com/iotaledger/trinity-wallet) The official wallet for IOTA with a user interface that allows you to transfer data and IOTA tokens

## Second layer protocols

Second layer protocols build on the IOTA protocol to offer extended functionality.

- [**Unified Identity Protocol:**](https://github.com/iotaledger/identity.ts) A protocol for giving things, individuals, and organizations control of their identities and personal data

- [**Masked Authenticated Messaging:**](https://github.com/iotaledger/mam.c) A protocol for sending encrypted streams of data over the Tangle

### Utilities

Utilities are open-source applications that you can either use to speed up development or adapt and include in your own applications.

- [Tangle utilities](https://github.com/iotaledger/tangle-utils-website)

- [Client load balancer](https://github.com/iotaledger/client-load-balancer)

- [Trytes compressor](https://github.com/iotaledger/tryte-compress-js)

- [IOTA area codes](https://github.com/iotaledger/iota-area-codes)

- [GitHub Tangle release](https://github.com/iotaledger/gh-tangle-release)

- [Proof of existence](https://github.com/iotaledger/iota-poex-tool)

### Blueprint demos

Blueprint demos are example applications of how IOTA technology can be used to solve real problems and to support well defined business needs.

- [**Tangle data storage:**](https://github.com/iotaledger/poc-ipfs) Allows users to verify the contents of a file in a third-party storage solution

- [Data Marketplace](https://github.com/iotaledger/data-marketplace) Allows users to pay for data in micropayments of IOTA tokens

- [Industry Marketplace](https://github.com/iotaledger/industry-marketplace) Allows devices to buy and sell goods, data, and services in a vendor-neutral marketplace 

- [HyperLedger bridge](https://github.com/iotaledger/HyperledgerFabric-IOTA-Connector) Allows users to connect to trigger transactions on the Tangle, using HyperLedger smart contracts

- [Container tracking](https://github.com/iotaledger/trade-poc) Allows users to track and trace assets in the supply chain
</details>

<br>

<details>
<summary>Getting started with your first contribution</summary>
<br>

At the root of each IOTAledger repository is a contribution guide with advice for getting started.

If you are unsure where to begin, you can start by looking any `beginner` or `help-wanted` issues.
</details>

<br>

<details>
<summary>Code of Conduct</summary>
<br>

This project and everyone participating in it is governed by the [IOTA Code of Conduct](../iotaledger/code-of-conduct.md).
</details>
