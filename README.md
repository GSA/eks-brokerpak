# eks-brokerpak

## Why this project

The EKS brokerpak is a
[cloud-service-broker](https://github.com/pivotal/cloud-service-broker) plugin
that makes AWS EKS brokerable via the [Open Service Broker API](https://www.openservicebrokerapi.org/) (compatible with Cloud Foundry and Kubernetes), using Terraform.

Making EKS brokerable has two drivers:
1. Some workloads don't work as an app/in a PaaS. Teams can broker a k8s "sidecar" and deploy just those parts, while retaining use of the PaaS and avoiding interaction with the IaaS.
2. Further brokerpaks can expose other services easily, using this brokerpak as a base. ([Here's a companion brokerpak that uses the Cloud Foundry Terraform provider to spin up a k8s instance via this broker, then deploys a Helm chart into it](https://github.com/GSA/datagov-brokerpak).)

For more information about the brokerpak concept, here's a [5-minute lightning
talk](https://www.youtube.com/watch?v=BXIvzEfHil0) from the 2019 Cloud Foundry Summit. You may also want to check out the brokerpak
[introduction](https://github.com/pivotal/cloud-service-broker/blob/master/docs/brokerpak-intro.md)
and
[specification](https://github.com/pivotal/cloud-service-broker/blob/master/docs/brokerpak-specification.md)
docs.

Huge props go to @josephlewis42 of Google for publishing and publicizing the
brokerpak concept, and to the Pivotal team running with the concept!


## Features/components

Each brokered AWS EKS provides:

- unique, isolated VPC with multiple AZs
- Kubernetes nodes provided by AWS EKS Fargate
- Bound credential permissions are limited by namespace
- cluster-wide logging to AWS CloudWatch with fluent-bit
- Automatic TLS configuration using AWS Certificate Manager (ACM) 
- Cluster-wide ingress load-balancing with AWS-specific features (eg WAFv2 integration) [via ALB Ingress Controller](https://kubernetes-sigs.github.io/aws-load-balancer-controller/latest/)
- nginx ingress controller for routing and IaaS-independent deployments
- Automatic DNS configuration using AWS Route53 [via ExternalDNS](https://github.com/kubernetes-sigs/external-dns)
- [ZooKeeper CRDs](https://github.com/pravega/zookeeper-operator) ready for
  managing Apache ZooKeeper clusters
- [Solr CRDs](https://github.com/apache/solr-operator) for managing
  SolrCloud instances


## Development Prerequisites

1. [Docker Desktop (for Mac or
Windows)](https://www.docker.com/products/docker-desktop) or [Docker Engine (for
Linux)](https://www.docker.com/products/container-runtime) is used for
building, serving, and testing the brokerpak.
1. [Access to the GitHub Container
   Registry](https://docs.github.com/en/packages/guides/migrating-to-github-container-registry-for-docker-images#authenticating-with-the-container-registry).
   (We are working on making the necessary container image publicly accessible;
   this step should not be necessary in future.)

1. `make` is used for executing docker commands in a meaningful build cycle. 
1. [`eden`](https://github.com/starkandwayne/eden) is used as a client for testing the brokerpak
1. Your AWS account credentials (as environment variables) are used for actual
   service provisioning. Set at least:
  * AWS_ACCESS_KEY_ID
  * AWS_SECRET_ACCESS_KEY
  * AWS_DEFAULT_REGION


## Developing the brokerpak 
Run the `make` command by itself for information on the various targets that are available. Notable targets are described below

```
$ make
clean      Bring down the broker service if it's up, clean out the database, and remove created images
build      Build the brokerpak(s) and create a docker image for testing it/them
up         Run the broker service with the brokerpak configured. The broker listens on `0.0.0.0:8080`. curl http://127.0.0.1:8080 or visit it in your browser.
test       Execute the brokerpak examples against the running broker
down       Bring the cloud-service-broker service down
all        Clean and rebuild, then bring up the server, run the examples, and bring the system down
help       This help
```


### Running the brokerpak
Run
```
make build up
```
The broker will start and listen on `0.0.0.0:8080`. You
test that it's responding by running:
```
curl -i -H "X-Broker-API-Version: 2.16" http://user:pass@127.0.0.1:8080/v2/catalog
```
In response you will see a YAML description of the services and plans available
from the brokerpak.

(Note that the `X-Broker-API-version` header is [**required** by the OSBAPI
specification](https://github.com/openservicebrokerapi/servicebroker/blob/master/spec.md#headers).
The broker will reject requests that don't include the header with `412
Precondition Failed`, and browsers will show that status as `Not Authorized`.)

You can also inspect auto-generated documentation for the brokerpak's offerings
by visiting [`http://127.0.0.1:8080/docs`](http://127.0.0.1:8080/docs) in your browser.

### Testing the brokerpak (while it's running)

Run 
```
make test
```

The [examples specified by the
brokerpak](https://github.com/pivotal/cloud-service-broker/blob/master/docs/brokerpak-specification.md#service-yaml-flie)
will be invoked for end-to-end testing of the brokerpak's service offerings.

You can also manually interact with the broker using the [`eden` OSBAPI client](https://github.com/starkandwayne/eden)

### Shutting the brokerpak down

Run 

```
make down
```

The broker will be stopped.

### Cleaning out the current state

Run 
```
make clean
```
The built brokerpak files will be removed.

## Contributing

Check
out the list of [open issues](https://github.com/GSA/eks-brokerpak/issues) for
areas where you can contribute.

See [CONTRIBUTING](CONTRIBUTING.md) for additional information. 

## Public domain

This project is in the worldwide [public domain](LICENSE.md). As stated in [CONTRIBUTING](CONTRIBUTING.md):

> This project is in the public domain within the United States, and copyright and related rights in the work worldwide are waived through the [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).
>
> All contributions to this project will be released under the CC0 dedication. By submitting a pull request, you are agreeing to comply with this waiver of copyright interest.
