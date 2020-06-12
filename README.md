[![Go Report Card](https://goreportcard.com/badge/github.com/mhmxs/calico-route-reflector-operator)](https://goreportcard.com/report/mhmxs/calico-route-reflector-operator) [![Build Status](https://travis-ci.org/mhmxs/calico-route-reflector-operator.svg?branch=master)](https://travis-ci.org/mhmxs/calico-route-reflector-operator) [![Docker Repository on Quay](https://quay.io/repository/mhmxs/calico-route-reflector-controller/status "Docker Repository on Quay")](https://quay.io/repository/mhmxs/calico-route-reflector-controller) [![Active](http://img.shields.io/badge/Status-Active-green.svg)](https://github.com/mhmxs/calico-route-reflector-operator) [![PR's Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat)](https://github.com/mhmxs/calico-route-reflector-operator/pulls) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

# Calico Route Reflector Operator

### Use your own risk !!!

### Proposal documentation found here: https://github.com/mhmxs/calico-route-reflector-operator-proposal. Please feel free to share your ideas !!!

## Prerequisites

 * Kubernetes cluster up and running
 * Calico network on Kubernetes data store a.k.a. `KDD`.
 * Configured Calico `BGPPeer`s one for route reflector mesh and an other for clients. [More info](https://docs.projectcalico.org/getting-started/kubernetes/hardway/configure-bgp-peering)

This Kubernetes operator can monitor and scale Calico route refloctor pods based on node number per zone. The operator owns a few environment variables:
 * `DATASTORE_TYPE` Calico datastore [`incluster`, `kubernetes`, `etcd`], default `incluster`
 * `K8S_API_ENDPOINT` Kubernetes API endpoint, default `https://kubernetes.default`
 * `ROUTE_REFLECTOR_CLUSTER_ID` Route reflector cluster ID, default `224.0.0.%d`
 * `ROUTE_REFLECTOR_MIN` Minimum number of route reflector pods per zone, default `3`
 * `ROUTE_REFLECTOR_MAX` Maximum number of route reflector pods per zone, default `25`
 * `ROUTE_REFLECTOR_RATIO` Node / route reflector pod ratio, default `0.005` (`1000 * 0.005 = 5`)
 * `ROUTE_REFLECTOR_NODE_LABEL` Node label of the route reflector nodes, default `calico-route-reflector=`
 * `ROUTE_REFLECTOR_ZONE_LABEL` Node label of the zone, default ``
 * `ROUTE_REFLECTOR_TOPOLOGY` Selected topology of route reflectors [simple, multi], default `simple`

You can edit or add those environment variables at the [manager](config/manager/manager.yaml) manifest. You can add Calico client config related variables, Calico lib will parse it in the background.

During the `api/core/v1/Node` reconcile phases it calculates the right number of route refloctor nodes per zone. It supports linear scaling only and it multiplies the number of nodes with the given ratio and than updates the route reflector replicas to the expected number. After all the nodes are labeled correctly it regenerates BGP peer topology for the cluster.

## Usage

This is a standard Kubebuilder opertor so building and deploying process is similar as a [stock Kubebuilder project](https://book.kubebuilder.io/cronjob-tutorial/running.html).

After first reconcile phase is done don not forget to disable the [node-to-node mesh](https://docs.projectcalico.org/getting-started/kubernetes/hardway/configure-bgp-peering)!

Use latest release:
```
kustomize build config/crd | kubectl apply -f -
$(cd config/manager && kustomize edit set image controller=quay.io/mhmxs/calico-route-reflector-controller:v0.0.2)
kustomize build config/default | kubectl apply -f -
```

Use official latest master image:
```
kustomize build config/crd | kubectl apply -f -
kustomize build config/default | kubectl apply -f -
```

Build your own image:
`IMG_REPO=[IMG_REPO] IMG_NAME=[IMG_NAME] IMG_VERSION=[IMG_VERSION] make test docker-push install deploy`

## Limitations

 * Multi cluster topology doesn't select route reflectors from multiple zones, so on case of full zone outages you have a chance some network outage if a node all 3 connections was established toward of that zone.
 * Multi cluster topology rebalances the whole cluster on case of nodes are added. If you are unlicky it could drop all 3 route reflector sessions which chause 1-2 sec network outage.
 

## Roadmap

 * Etcd data store support (You have to add Etcd related env vars into [manager](config/manager/manager.yaml) manually)
 * Use custom resource instead of environment variables
 * Dedicated or preferred node label
 * Disallow node label
 * Handle taints and tolerations

# Contributing

We appreciate your help!

To contribute, please read our contribution guidelines: [CONTRIBUTION.md](CONTRIBUTION.md)
