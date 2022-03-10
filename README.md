# idmz-helm

Helm Chart for IDMZ Deployments

## TL;DR

```json
helm repo add cybertrol-engineering-idmz https://cybertrol-engineering.github.io/idmz-helm
helm install my-release cybertrol-engineering-idmz/idmz
```

## Introduction

This chart bootstraps an IDMZ deployment on a Kubernetes cluster using the Helm package manager.

## Prerequisites

- Kubernetes 1.22+

- Helm 3

- LoadBalancer service provisioner

- PV provisioner

- bitnami-labs/sealed-secretes 

- cert-manager/cert-manager

- stakater/Reloader

## Installing the Chart

```bash
helm install my-release cybertrol-engineering-idmz/idmz
```

## Uninstalling the Chart

```bash
helm delete my-release
```

## Parameters

### Global parameters

| Name          | Description                                            | Value     |
|---------------|--------------------------------------------------------|-----------|
|global.