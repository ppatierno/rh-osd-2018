# AAMQ Streams on OpenShift demo

AMQ Streams on OpenShift demo for the Red Hat Open Source Day 2018 (Milan and Rome).

## Installing the Cluster Operator

The Cluster Operator is the most important component in AMQ Streams on OpenShift.
It follows the usual Operator pattern of watching for changes on some desired resource and making changes in other resources to make them match the desired state.
It can be deployed like this:

    oc apply -f 01-install-cluster-operator/

Other then the Cluster Operator itself, a bunch of CRDs (Custom Resource Definitions) will be created as well in order to handle the Kafka cluster with related Custom Resources:

* `Kafka` : a Kafka cluster (with Zookeeper ensemble)
* `KafkaConnect` : a Kafka Connect cluster
* `KafkaConnectS2I` : a Kafka Connect cluster with OpenShift S2I support for adding plugins connectors
* `KafkaMirrorMaker` : a Kafka Mirror Maker replica
* `KafkaTopic` : a Kafka topic
* `KafkaUser`: a Kafka user (with related ACLs)

## Deploying Kafka cluster

The Kafka cluster is described by a `Kafka` custom resource in a corresponding YAML file:

    cat 02-kafka-cluster.yaml

It can be deployed like this:

    oc apply -f 02-kafka-cluster.yaml

Because the Kafka cluster, which is going to be deployed, exposes metrics about the cluster itself (both Zookeeper and Kafka), a Prometheus and Grafana servers are deployed first for gathering these metrics and showing them.

    oc apply -f 03-prometheus.yaml
    oc apply -f 04-grafana.yaml

The datasource configuration and Kafka and Zookeeper dashboards can be deployed running:

    ./05-grafana.sh



