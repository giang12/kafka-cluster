# Docker Kafka Cluster

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

* 1 kafka manager @ port `2190`
* N kafka connect @ port `808X` (starting from port 8083) 
* N kafka brokers @ port `209X` (starting from port 2091)
* N kafka exporters @ port `930X` (starting from port 9308)
* N zookeeper nodes @ port `218X` (starting from port 2181)

The script will automatically deploy kafka exporters, kafka connectors,
zookeeper nodes and kafka brokers depending upon the constrainsts specified.
The script will deploy one kafka broker to each instance with the constraint
`kafka=yes`, one exporter to each instance with `exporter=yes`, one kafka
connector to each instance with `connect=yes`, and one zookeeper node to each
instance with `zookeeper=yes`.

Each broker service will contain both a zookeeper and a kafka node. The
zookeeper will be available on the ports `218X` and the kafka broker will
be available on the ports `209X`.

## Running

Bring up the cluster. The `cluster-gen` script will automatically detect how
many nodes the swarm has with the `label.kafka == 'yes'`. There will be
a broker for each of kafka node it finds in the swarm.

```
./cluster-gen
./cluster-up
```

Sometimes the files will not be in the correct format, so you will need to use
`dos2unix`. Convert all the files recursively with

```
find . -type f -print0 | xargs -0 dos2unix
```

This will create a `kafka-compose.yml` file in the directory that will be used
by `./cluster-down` do delete the cluster (i.e. this stops and removes all data).
If you do not want to delete data, use the `docker-compose` commands directly.

To export data from kafka to ElasticSearch, use

```
./cluster-connect <connector-name> <kafka-topic>
```

Even if the cluster goes down, the kafka connector will persist and attempt to
continue to run after restarting the stack. The connector, however, will no longer
be assigned a leader. In order to correct this, you may have to delete kafka
connector directly from the kafka cluster using

```
curl -X DELETE [kafka connect url]:8083/connectors/[connector name]
```

## Kafka Opts

- KAFKA_ADVERTISED_HOST_NAME
- KAFKA_ADVERTISED_PORT
- KAFKA_BROKER_ID
- KAFKA_LOG_DIRS
- KAFKA_LOG_RETENTION_HOURS
- KAFKA_LOG_RETENTION_BYTES
- KAFKA_ZOOKEEPER_CONNECT

## Zookeeper Opts

- ZOO_ID
- ZOO_DATA_DIR
- ZOO_DATA_LOG_DIR
- ZOO_LOG_DIR
- ZOO_PORT
- ZOO_TICK_TIME
- ZOO_INIT_LIMIT
- ZOO_SYNC_LIMIT
- ZOO_AUTOPURGE_PURGEINTERVAL
- ZOO_AUTOPURGE_SNAPRETAINCOUNT
- ZOO_MAX_CLIENT_CNXNS

## Automatically create topics

If you want to have kafka-docker automatically create topics in Kafka during
creation, edit ```KAFKA_CREATE_TOPICS``` in `cluster-gen`

Here is an example snippet from ```docker-compose.yml```:

          KAFKA_CREATE_TOPICS="Topic1:1:3,Topic2:1:1:compact"

```Topic 1``` will have 1 partition and 3 replicas, ```Topic 2``` will have 1 partition, 1 replica and a `cleanup.policy` set to `compact`. Also, see FAQ: [Topic compaction does not work](https://github.com/wurstmeister/kafka-docker/wiki#topic-compaction-does-not-work)

## Connectivity

The cluster will contain ZooKeeper nodes accessible as `localhost:2181,`
`localhost:2182,...`. The brokers will be reachable under
`localhost:2091,localhost:2092,...` and advertise Docker host IP to Kafka
clients. As this is reachable from the host and from the Docker network, it is
possible to connect to the cluster both from the host machine as well as from
other Docker containers.

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
