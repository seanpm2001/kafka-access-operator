[![Build Status](https://dev.azure.com/cncf/strimzi/_apis/build/status%2Faccess-operator%2Faccess-operator?branchName=main)](https://dev.azure.com/cncf/strimzi/_build/latest?definitionId=51&branchName=main)
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Twitter Follow](https://img.shields.io/twitter/follow/strimziio?style=social)](https://twitter.com/strimziio)

# Access Operator for Strimzi

This project provides a Kubernetes operator to help applications bind to an [Apache Kafka®](https://kafka.apache.org) cluster that is managed by the [Strimzi](https://strimzi.io) cluster operator.

The operator creates a single Kubernetes `Secret` resource containing all the connection details for the Kafka cluster.
The removes the need for applications to query multiple Kubernetes resources to get connection information.
The `Secret` follows the conventions laid out in the [Service Binding Specification for Kubernetes v1.0.0](https://servicebinding.io/spec/core/1.0.0/).

The operator is built using the [Java Operator SDK](https://github.com/java-operator-sdk/java-operator-sdk).

## Running the Access Operator

The Access Operator is still in early stages of development so to run it you need to build it from source.
The [dev guide](https://github.com/strimzi/kafka-access-operator/blob/main/development-docs/DEV_GUIDE.md) describes how to build and run the Access Operator.

To start the operator you need the Strimzi `Kafka` and `KafkaUser` custom resource definitions installed in your Kubernetes cluster.
You can get these from the Strimzi [GitHub repository](https://github.com/strimzi/strimzi-kafka-operator/tree/main/packaging/install/cluster-operator),
or use the [Strimzi quickstart guide](https://strimzi.io/quickstarts/) to also deploy the Strimzi cluster operator and a Kafka instance at the same time.

## Using the Access Operator

To make use of the Access Operator, create a `KafkaAccess` custom resource (CR).
You must specify the name of the name of the `Kafka` CR you want to connect to.
You can optionally also specify the name of the listener in the `Kafka` CR and a `KafkaUser`.
See the [examples folder](https://github.com/strimzi/kafka-access-operator/tree/main/packaging/examples) for some valid `KafkaAccess` specifications.

If you do not specify which listener you want to connect to, the operator uses the following rules to choose a listener:
1. If there is only one listener configured in the `Kafka` CR, that listener is chosen.
2. If there are multiple listeners listed in the `Kafka` CR, the operator filters the list by comparing the `tls` and `authentication` properties in the `Kafka` and `KafkaUser` CRs to select a listener with the appropriate security.
3. If there are multiple listeners with appropriate security, the operator chooses the one that is of type `internal`.
4. If there are multiple internal listeners with appropriate security, the operator sorts the listeners alphabetically by name, and chooses the first one.

Once the Access Operator has created the binding `Secret`, it updates the `KafkaAccess` custom resource to put the name of the secret in the status, for example:

```yaml
...
status:
  binding:
    name: kafka-binding
```

The `Secret` created by the Access Operator has the following structure:

```yaml
apiVersion: v1
kind: Secret
metadata:
    name: kafka-binding
type: servicebinding.io/kafka
data:
    type: kafka
    provider: strimzi

    bootstrap.servers: # comma separated list of host:port for Kafka
    bootstrap-servers: # comma separated list of host:port for Kafka
    bootstrapServers: # comma separated list of host:port for Kafka

    security.protocol: # one of PLAINTEXT, SASL_PLAINTEXT, SASL_SSL or SSL
    securityProtocol: # one of PLAINTEXT, SASL_PLAINTEXT, SASL_SSL or SSL

    # Provided if TLS enabled:
    ssl.truststore.crt: #  Strimzi cluster CA certificate

    # Provided if selected user is SCRAM auth:
    username: # SCRAM username
    password: # SCRAM password
    sasl.jaas.config: # sasl jaas config string for use by Java applications
    sasl.mechanism: SCRAM-SHA-512
    saslMechanism: SCRAM-SHA-512

    # Provided if selected user is mTLS:
    ssl.keystore.crt: # certificate for the consuming client signed by the clients' CA
    ssl.keystore.key: # private key for the consuming client
```

Developers can make this `Secret` available to their applications themselves, or use an operator that implements the [Service Binding specification](https://servicebinding.io/spec/core/1.0.0/) to do it.

## Getting help

If you encounter any issues while using the Access Operator, you can get help through the following methods:

- [Strimzi Users mailing list](https://lists.cncf.io/g/cncf-strimzi-users/topics)
- [#strimzi channel on CNCF Slack](https://slack.cncf.io/)
- [GitHub Discussions](https://github.com/orgs/strimzi/discussions)

## Contributing

You can contribute by:
- Raising any issues you find using the Access Operator
- Fixing issues by opening Pull Requests
- Improving documentation
- Talking about the Strimzi Access Operator

All bugs, tasks or enhancements are tracked as [GitHub issues](https://github.com/strimzi/kafka-access-operator/issues).

The [dev guide](https://github.com/strimzi/kafka-access-operator/blob/main/development-docs/DEV_GUIDE.md) describes how to build the operator and how to test your changes before submitting a patch or opening a PR.

If you want to get in touch with us first before contributing, you can use:

- [Strimzi Dev mailing list](https://lists.cncf.io/g/cncf-strimzi-dev/topics)
- [#strimzi channel on CNCF Slack](https://slack.cncf.io/)

Learn more on how you can contribute on our [Join Us](https://strimzi.io/join-us/) page.

## License

Strimzi Access Operator is licensed under the [Apache License](./LICENSE), Version 2.0
