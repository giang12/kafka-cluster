FROM anapsix/alpine-java

MAINTAINER Zen Nguyen, https://github.com/giang12

# Versions
ARG scala_version=2.12
ARG kafka_version=1.1.0

ENV KAFKA_VERSION=$kafka_version \
    SCALA_VERSION=$scala_version \
    KAFKA_HOME=/kafka \
    GLIBC_VERSION=$glibc_version

# Fetch Kafka & Zookeeper tarfile from apache mirror & extract it to /kafka_<versions-info>.
RUN \
    echo "===> fetching Kafka..."  && \
    wget -q http://mirror.23media.de/apache/kafka/${kafka_version}/kafka_${SCALA_VERSION}-${KAFKA_VERSION}.tgz -O /tmp/kafka.tgz && \
    \
    \
    echo "===> installing Kafka..."  && \
    tar -C / -xzf /tmp/kafka.tgz && \
    ln -s /kafka_${SCALA_VERSION}-${KAFKA_VERSION} /kafka && \
    \
    \
    echo "===> clean up..."  && \
    rm /tmp/kafka.tgz

# Add Kafka & Zookeeper custom configurations.
ADD files/config/server.properties files/config/zookeeper.properties files/config/log4j.properties /kafka/config/

# Add Kafka & Zookeeper start wrapper.
ADD files/bin/start \
    files/bin/start-kafka-shell.sh \
    files/bin/update_kafka_config \
    files/bin/broker-list.sh \
    files/bin/create-topics.sh \
    /usr/bin/

RUN chmod -R a+x /usr/bin/ && export PATH=$PATH:/usr/bin

# Expose config volume.
VOLUME /kafka/config

# Expose Kafka data volume.
VOLUME /var/lib/kafka

# Kafka broker port.
ENV KAFKA_ADVERTISED_HOST_NAME=127.0.0.1

# Kafka broker port.
ENV KAFKA_ADVERTISED_PORT=9092

# Default Kafka broker id
ENV KAFKA_BROKER_ID=1

# Define start wrapper as entrypoint.
ENTRYPOINT [ "/usr/bin/start" ]

# Define kafka as default command (the other option being "zookeeper").
CMD [ "kafka" ]
