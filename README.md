# Local Docker Kafka Cluster

[![docker pulls](https://img.shields.io/docker/pulls/zen12/kafka-cluster.svg)](https://hub.docker.com/r/zen12/kafka-cluster/)
[![docker stars](https://img.shields.io/docker/stars/zen12/kafka-cluster.svg)](https://hub.docker.com/r/zen12/kafka-cluster/)

Creates a [Docker](https://www.docker.com)-based [Kafka](http://kafka.apache.org/)
cluster with [docker-compose](https://docs.docker.com/compose) for local
development of Kafka-based apps.

## Prerequisites

Works for Linux and Mac (with the latest Docker for Mac), and uses docker
compose file syntax version 2.

- [Docker](https://www.docker.com) 1.10.0 or later
- [Docker Compose](https://docs.docker.com/compose) 1.6.0 or later
- [Ruby](https://www.ruby-lang.org)

## In the Box

1 zookeeper @ port `2181`

1 kafka manager @ port `2190`

Also, the ports `2091`, `2092`, and `2093` (for a default three broker
setup) should be available.

## Running

Bring up the cluster and specify that you want e.g. five brokers (the default
cluster size is three brokers) with

```
./cluster-gen 5
./cluster-up
```

This will create a `kafka-compose.yml` file in the directory that will be used
by `./cluster-down` do delete the cluster (i.e. this stops and removes all data).
If you do not want to delete data, use the `docker-compose` commands directly.
## Kafka Opts

- KAFKA_ADVERTISED_HOST_NAME
- KAFKA_ADVERTISED_PORT
- KAFKA_BROKER_ID
- KAFKA_LOG_DIRS
- KAFKA_LOG_RETENTION_HOURS
- KAFKA_LOG_RETENTION_BYTES
- KAFKA_ZOOKEEPER_CONNECT

## Connectivity

The cluster will contain one ZooKeeper node only accessible as `localhost:2181`.
The brokers will be reachable under `localhost:2091,localhost:2092,...` and
advertise Docker host IP to Kafka clients. As this is reachable from the host
and from the Docker network, it is possible to connect to the cluster both
from the host machine as well as from other Docker containers.

## The Thing With the Docker Network and Kafka

Kafka Brokers advertise themselves to each other and clients via the Kafka
protocol, and announce hostname/IP and port. It is therefore necessary to have
fixed (distinct) ports, and have a hostname or IP that is resolvable or routable
from both the docker network (for broker-to-broker communication) and from the
docker host (for local development).

To achieve this for both Linux and Mac - in the face of
[some verified existing issues](https://github.com/docker/docker/issues/22753)
with Mac - the best way to go is to use the [docker host IP](./cluster-up#L10-L15).
Without that issue, it would be more sensible to use the docker container IPs.

Using the docker host IP as advertised broker address makes broker-to-broker
traffic pass through the docker host, but this overhead is considered an
acceptable price for a solution working both on Linux and Mac.
