# AAMQ Streams on OpenShift demo

AMQ Streams on OpenShift demo for the Red Hat Open Source Day 2018 (Milan and Rome).

## Installing the Cluster Operator

The Cluster Operator is the most important component in AMQ Streams on OpenShift.
It follows the usual Operator pattern of watching for changes on some desired resource and making changes in other resources to make them match the desired state.
It can be deployed like this:

    oc apply -f 01-install-cluster-operator/
