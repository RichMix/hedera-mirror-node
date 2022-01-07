[![Build](https://github.com/hashgraph/hedera-mirror-node/actions/workflows/importer.yml/badge.svg)](https://github.com/hashgraph/hedera-mirror-node/actions)
[![codecov](https://img.shields.io/codecov/c/github/hashgraph/hedera-mirror-node/main)](https://codecov.io/gh/hashgraph/hedera-mirror-node)
[![Discord](https://img.shields.io/badge/discord-join%20chat-blue.svg)](https://hedera.com/discord)

# Hedera Mirror Node

The Hedera Mirror Node acts as an archive node and stores historical data for the Hedera network.

## Overview

Mirror nodes receive information from the Hedera nodes and can provide value-added services such as APIs, auditing,
analytics, visibility services, security threat modeling, data monetization services, etc. Mirror nodes can also run
additional business logic to support applications built using the Hedera network.

While mirror nodes receive information from the main nodes, they do not contribute to consensus on the network, and
their votes are not counted. Only the votes from the main nodes are counted for determining consensus. The trust of the
Hedera network is derived based on the consensus reached by the main nodes. That trust is transferred to the mirror
nodes using cryptographic signatures on a chain of files.

Eventually, the mirror nodes will be able to run the same code as the Hedera nodes so that they can see the transactions
in real time. To make the initial deployments easier, the mirror node strives to take away the burden of running a full
Hedera node through creation of periodic files that contain processed information (such as account balances or
transaction records), and have the full trust of the main nodes. The mirror node software reduces the processing burden
by receiving pre-constructed files from the network, validating those, populating a database, and providing APIs to
expose the data. This approach provides the following advantages:

- Lower compute and bandwidth requirements
- Allows users to only save the data that they care about (lower storage requirement)
- Easy searchable database so the users can add value quickly
- Easy to consume APIs to make integrations faster

## Architecture

### Main Nodes

- When a transaction reaches consensus, Hedera nodes add the transaction and its associated record to a
  [record file](https://docs.hedera.com/guides/docs/record-and-event-stream-file-formats#version-5-record-stream-file-format)
  .
- Record files contain the hash of the previous record file, thus creating an unbreakable validation chain.
- The file is closed on a regular cadence and a signature file is generated by the node for the record file.
- The record and signature files from each node are then uploaded to Amazon S3 and Google Cloud Storage.

### Mirror Nodes

- This mirror node software downloads signature files from cloud storage.
- The signature files are verified using the corresponding node's public key from the address book (stored in
  a `0.0.102` file)
- The verified signature files are checked to ensure at least 1/3 have the same record file hash
- For each valid signature file, the corresponding record file is then downloaded from cloud storage and its hash
  verified against the hash contained in the signature file.
- The downloaded record file contains a previous hash that is validated against the last processed file to verify the
  hash chain.
- Record files can then be processed and transactions and records processed for long term storage.

In addition, nodes regularly generate
a [balance file](https://github.com/hashgraph/hedera-protobufs/blob/main/streams/account_balance_file.proto) and an
[event file](https://docs.hedera.com/guides/docs/record-and-event-stream-file-formats#version-5-event-stream-file-format)
. The balance file contains the list of Hedera accounts and their corresponding HBAR and token balances. The event file
contains events that were gossiped between nodes. Both of these files are processed by the mirror node in a similar
fashion as outlined above.

## Getting Started

### Technologies

Multiple technologies are utilized in the mirror node. The following topics are areas where basic knowledge is valuable
to understanding the mirror node operations.

- [Git](https://git-scm.com/about)
- [Go](https://golang.org/)
- [Docker](https://docs.docker.com/engine/reference/commandline/docker/)
- [Helm](https://helm.sh)
- [Maven](https://maven.apache.org/guides/getting-started/index.html)
- [Node.js](https://nodejs.org/en/about/)
- [PostgreSQL](https://www.postgresql.org/docs)
- [Spring Boot](https://docs.spring.io/spring-boot/docs/current/reference/html/getting-started.html#getting-started)

### Prerequisite Tools

Ensure these tools are installed (note minimum versions) prior to running the mirror node:

- [Docker Desktop 4.3+](https://www.docker.com/products/docker-desktop)

### Running

For production use, we recommend using Kubernetes and to deploy using our Helm [chart](charts). Hedera managed mirror
nodes use Kubernetes and Helm for their deployments, and this process is considered the most production-ready. As an
alternative for local development, Docker Compose can be used to run the mirror node. See
the [installation](docs/installation.md#running-via-docker-compose) document for more details on configuring and running
with Docker Compose. To get up and running quickly with Docker Compose execute the following commands in your terminal:

```bash
git clone https://github.com/hashgraph/hedera-mirror-node.git
cd hedera-mirror-node
docker compose up
```

> **_NOTE:_** This defaults to a bucket setup for demonstration purposes. See the next section for more details.

## Data Access

### Demo

The free option utilizes a bucket setup for demonstration purposes. This is not a real Hedera network but simply a dummy
bucket populated with a day's worth of past testnet data. This is the default option and requires no additional steps.
Once you've verified your deployment works against the demo bucket, remember to configure it with
a [public network](#public-networks) then wipe the database and restart the mirror node.

### Public Networks

To access data from real Hedera networks,
[AWS](https://docs.aws.amazon.com/AmazonS3/latest/dev/RequesterPaysBuckets.html) or
[GCS](https://cloud.google.com/storage/docs/requester-pays) requester pays credentials must be used. The charges
associated with the downloading of stream files are paid for by the requester and not the bucket owner.

## Documentation

- Components
  - [gRPC API](docs/grpc/README.md)
  - [Monitor](docs/monitor/README.md)
  - [REST API](docs/rest/README.md)
  - [Rosetta API](docs/rosetta/README.md)
  - [Web3 API](docs/web3/README.md)
- [Installation](docs/installation.md)
- [Configuration](docs/configuration.md)
- [Operations](docs/operations.md)
- [Troubleshooting](docs/troubleshooting.md)

## Releasing

To perform a new release, run the `Automated Release` GitHub workflow.

## Support

If you have a question on how to use the product, please see our
[support guide](https://github.com/hashgraph/.github/blob/main/SUPPORT.md).

## Contributing

Contributions are welcome. Please see the
[contributing guide](https://github.com/hashgraph/.github/blob/main/CONTRIBUTING.md)
to see how you can get involved.

## Code of Conduct

This project is governed by the
[Contributor Covenant Code of Conduct](https://github.com/hashgraph/.github/blob/main/CODE_OF_CONDUCT.md). By
participating, you are expected to uphold this code of conduct. Please report unacceptable behavior
to [oss@hedera.com](mailto:oss@hedera.com).

## License

[Apache License 2.0](LICENSE)
