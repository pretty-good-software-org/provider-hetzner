<!--
SPDX-FileCopyrightText: 2023 The Crossplane Authors <https://crossplane.io>

SPDX-License-Identifier: CC-BY-4.0
-->

# Upjet-based Crossplane provider for Hetzner

<div style="text-align: center;">

![CI](https://github.com/miaits/provider-hetzner/workflows/CI/badge.svg)
[![GitHub release](https://img.shields.io/github/release/miaits/provider-hetzner/all.svg)](https://github.com/miaits/provider-hetzner/releases)
[![Go Report Card](https://goreportcard.com/badge/github.com/miaits/provider-hetzner)](https://goreportcard.com/report/github.com/miaits/provider-hetzner)
[![Contributors](https://img.shields.io/github/contributors/miaits/provider-hetzner)](https://github.com/miaits/provider-hetzner/graphs/contributors)

</div>

Provider Hetzner is a [Crossplane](https://crossplane.io/) provider that is built using
[Upjet](https://github.com/crossplane/upjet) code generation tools and exposes XRM-conformant managed resources for
[Hetzner Cloud](https://www.hetzner.com/cloud).

## Getting Started

The provider needs a Kubernetes secret with an API Token for Hetzner Cloud. To create the secret, run:

```bash
kubectl create secret generic hetzner   \
--from-literal=credentials='{"token":"<TOKEN>"}'   \
--dry-run=client -o yaml | kubectl apply -f -
```

To install the provider into a local Kubernetes cluster with Crossplane already installed, apply:

```yaml
apiVersion: pkg.crossplane.io/v1
kind: Provider
metadata:
  name: provider-hetzner
spec:
  package: ghcr.io/miaits/provider-hetzner:v1.0.0-alpha.1
```

To create a Hetzner server for test, apply:

```yaml
apiVersion: server.hetzner.m.crossplane.io/v1alpha1
kind: Server
metadata:
  annotations:
    meta.upbound.io/example-id: server/v1alpha1/server
  labels:
    testing.upbound.io/example-name: node1
  name: node1
spec:
  forProvider:
    image: debian-12
    name: node1
    serverType: cx23
```

To delete the test server, run:

```bash
kubectl delete servers.server.hetzner.m node1
```

## Contributing

For the general contribution guide, see
[Upjet Contribution Guide](https://github.com/crossplane/upjet/blob/main/CONTRIBUTING.md)

If you'd like to learn how to use Upjet, see [Usage Guide](https://github.com/crossplane/upjet/tree/main/docs).

To build this provider locally and run it in a local Kubernetes cluster, run `make run`

### Add a New Resource

Follow the Upjet guide for
[adding new resources](https://github.com/crossplane/upjet/blob/main/docs/adding-new-resource.md).

## Getting help

For filing bugs, suggesting improvements, or requesting new resources or features, please open an
[issue](https://github.com/miaits/provider-hetzner/issues/new/choose).

## License

The provider is released under the [the Apache 2.0 license](LICENSE) with [notice](NOTICE).

## Markdown formatter support

The repository-local mdformat lifecycle is POSIX-only. It supports Linux ARM64, Linux x64 including glibc and musl, and
macOS ARM64 and x64. Windows is not supported because the canonical Mise locks intentionally contain no Windows
artifacts.
