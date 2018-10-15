# AMQ Streams on OpenShift demo

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
* `KafkaMirrorMaker` : a Kafka Mirror Maker deployment
* `KafkaTopic` : a Kafka topic
* `KafkaUser`: a Kafka user (with related ACLs)

## Deploying Kafka cluster

The Kafka cluster is described by a `Kafka` custom resource in a corresponding YAML file:

    cat 02-kafka-cluster.yaml

It can be deployed like this:

    oc apply -f 02-kafka-cluster.yaml

It deployes a Kafka cluster (alongside with a Zookeeper ensemble) and two other operators:

* Topic Operator: for handling Kafka topics (creation, deletion, ...) via a `KafkaTopic` resource
* User Operator: for handling Kafka users for authentication and related ACLs (Access Control Lists) via a `KafkaUser` resource

Because the Kafka cluster, which is going to be deployed, exposes metrics about the cluster itself (both Zookeeper and Kafka), a Prometheus and Grafana servers can be deployed for gathering these metrics and showing them.

    oc apply -f 03-prometheus.yaml
    oc apply -f 04-grafana.yaml

The datasource configuration and Kafka and Zookeeper dashboards can be deployed running:

    ./05-grafana.sh

> Prometheus and Grafana aren't part of AMQ Streams. The above is just an example of a possible dashboard.

## Kafka topic and user creation

A Kafka topic can be created in the cluster by creating `KafkaTopic` resource.
The Topic Operator is watching for this kind of resources and then creates the related Kafka topic.

    cat 06-kafka-topic.yaml

It can be created like this: 

    oc apply -f 06-kafka-topic.yaml

It is possible to check topic creation running the `kafka-topics.sh` tool on one of the broker pods:

    oc exec -it my-cluster-kafka-0 -c kafka -- bin/kafka-topics.sh --zookeeper localhost:2181 --describe

Getting an output describing the created topic with all the partitions and replicas.

```
Topic:my-topic  PartitionCount:3        ReplicationFactor:3     Configs:segment.bytes=1073741824,retention.ms=7200000
        Topic: my-topic Partition: 0    Leader: 1       Replicas: 1,0,2 Isr: 1,0,2
        Topic: my-topic Partition: 1    Leader: 2       Replicas: 2,1,0 Isr: 2,1,0
        Topic: my-topic Partition: 2    Leader: 0       Replicas: 0,2,1 Isr: 0,2,1
```

The demo applications use a user for authenticating agains the Kafka cluster, on which the TLS authentication is enabled.
A Kafka user with related ACLs can be created in the cluster by creating a `KafkaUser` resource.
The User Operator is watching for this kind of resources and then creates the related Kafka user.

    cat 07-kafka-user.yaml

It can be created like this:

    oc apply -f 07-kafka-user.yaml

For TLS authentication, the User Operator will create an X509 certificate for that user and sign it using an internal client Certificate Authority. The client also needs to trust the broker's certificates, so we need to ensure the certificate of the CA which signed the broker certificate is in the clients' trust stores.
Other than that, the User Operator creates the related ACL rules on Zookeeper.
It is possible to check it running a shell on one of the Zookeeper nodes first.

    oc exec -it my-cluster-zookeeper-0 -c zookeeper -- bin/zookeeper-shell.sh localhost:21810

Then getting the information from the corresponding znode.

    get /kafka-acl/Topic/my-topic

Getting an output describing what was defined in the `KafkaUser` resource.

```
{"version":1,"acls":[{"principal":"User:CN=my-user","permissionType":"Allow","operation":"Read","host":"*"},{"principal":"User:CN=my-user","permissionType":"Allow","operation":"Describe","host":"*"},{"principal":"User:CN=my-user","permissionType":"Allow","operation":"Write","host":"*"}]}
```

## Deploying demo producer/consumer applications

Producer and consumer applications can be deployed as following:

    oc apply -f 08-demo-application.yaml

Both clients use the same created user in order to authenticate to the Kafka cluster over TLS.
They exchange messages through the created topic.