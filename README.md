# idmz-helm
 Helm Chart for IDMZ Deployments

This documentation has been tested with microk8s. MetalLB is used as the load balancer.

```bash
microk8s enable storage
microk8s enable metallb
microk8s enable dns
sudo systemctl enable iscsid
microk8s enable openebs
```

## Environment Setup

### Install the dependencies
Note: These are not installed automatically.

```bash
helm repo add sealed-secrets https://bitnami-labs.github.io/sealed-secrets
helm repo add stakater https://stakater.github.io/stakater-charts
helm repo add cert-manager https://charts.jetstack.io
```

```bash
helm install cert-manager jetstack/cert-manager --namespace cert-manager --create-namespace --set installCRDs=true --version v1.7.0
```

```bash
helm install sealed-secrets sealed-secrets/sealed-secrets --namespace sealed-secrets --set installCRDs=true --create-namespace --version 2.1.2
```

```bash
helm install reloader stakater/reloader --namespace reloader --create-namespace --set installCRDs=true
```

### Generate Encrypted Secrets

```bash
echo -n "yoursecret" | kubeseal --controller-namespace sealed-secrets --raw --scope cluster-wide --from-file=/dev/stdin --controller-name sealed-secrets
```

Place these encrypted secrets into your values.yaml.

## Add the repo and install Zabbix

```bash
helm repo add alphabet5-idmz https://alphabet5.github.io/idmz-helm
helm install idmz alphabet5-idmz/idmz
```

## Installing With Flux

### Install Flux

```bash
flux install
```

### Generate a personal access token and export the value

```bash
export GITHUB_TOKEN=<your token here>
```

### Add your cluster's git repo

```bash
flux create source git idmz-deployment --url=https://github.com/alphabet5/idmz-deployment --branch=main --username=alphabet5-bot --password=$GITHUB_TOKEN
```

alphabet5-bot is a user with access only to this specific repository. Options are limited with https access.

Alternatively, you can create a deployment key and add the repository that way.

https://fluxcd.io/docs/cmd/flux_create_source_git/


Along with the flux manifests (`flux install --export`) here is an example:

```yaml
---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: sealed-secrets
  namespace: flux-system
spec:
  interval: 1h0m0s
  url: https://bitnami-labs.github.io/sealed-secrets

---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: stakater
  namespace: flux-system
spec:
  interval: 1h0m0s
  url: https://stakater.github.io/stakater-charts

---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: sealed-secrets
  namespace: flux-system
spec:
  interval: 1h0m0s
  url: https://charts.jetstack.io

---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: sealed-secrets
  namespace: flux-system
spec:
  chart:
    spec:
      chart: sealed-secrets
      sourceRef:
        kind: HelmRepository
        name: sealed-secrets
      version: '>=2.1.2'
  install:
    crds: Create
  interval: 1h0m0s
  releaseName: sealed-secrets
  targetNamespace: sealed-secrets
  upgrade:
    crds: CreateReplace

---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: reloader
  namespace: flux-system
spec:
  chart:
    spec:
      chart: reloader
      sourceRef:
        kind: HelmRepository
        name: stakater
  install:
    crds: Create
  interval: 1h0m0s
  releaseName: sealed-secrets-controller
  targetNamespace: reloader
  upgrade:
    crds: CreateReplace

---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: cert-manager
  namespace: flux-system
spec:
  chart:
    spec:
      chart: sealed-secrets
      sourceRef:
        kind: HelmRepository
        name: sealed-secrets
      version: '>=1.7.0'
  install:
    crds: Create
  interval: 1h0m0s
  targetNamespace: sealed-secrets
  upgrade:
    crds: CreateReplace

---
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: alphabet5-idmz
  namespace: flux-system
spec:
  interval: 1m0s
  url: https://alphabet5.github.io/idmz-helm
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: idmz
  namespace: idmz
spec:
  interval: 1m0s
  chart:
    spec:
      chart: idmz
      sourceRef:
        kind: HelmRepository
        name: alphabet5-idmz
  values:
    ## Your unique values.yaml for the idmz helm chart go here ##
```

### Add the Kustomization 

```bash
flux create kustomization idmz --source=idmz-deployment --path="./clusters/production" --prune=true --interval=1m
```

- Monitor the progress

```bash
watch flux get all
```

Flux seems to fail installing CRDs for cert-manager. If this happens, you can manually install the CRDs.

```bash
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.7.1/cert-manager.crds.yaml
```

