# Passbolt

[Passbolt](https://www.passbolt.com/) is an open sourced password manager.

## Introduction

This chart bootstraps a Passbolt instance.

## Prerequisites

- Kubernetes 1.14+

## Installing the chart

To install the chart:

```bash
$ helm repo add github-cnieg https://cnieg.github.io/helm-charts
$ helm install github-cnieg/passbolt
```

## Uninstalling the chart

To uninstall/delete the deployment:

```bash
$ helm list
NAME           REVISION    UPDATED                     STATUS      CHART              NAMESPACE
kindly-newt    1           Mon Oct  5 11:08:43 2020    DEPLOYED    passbolt-1.0.0    default
$ helm delete kindly-newt
```

## Configuration

The following table lists the configurable parameters of the Passbolt chart and their default values.

| Parameter                       | Description                                              | Default                     |
| ------------------------------- | -------------------------------------------------------- | --------------------------- |
| `image.repository`              | image repository                                         | `passbolt`                  |
| `image.tag`                     | `passbolt` image tag.                                    | `2.13.1-debian`             |
| `image.pullPolicy`              | Image pull policy                                        | `IfNotPresent`              |
| `imagePullSecrets`              | imagePullSecrets to use for private repositories         | `[]`                        |
| `nameOverride`                  | Override chart name                                      | ""                          |
| `fullnameOverride`              | Overrides Chart fullname                                 | ""                          |
| `podAnnotations`                | Custom annotations for pods                              | None                        |
| `podSecurityContext`            | Privilege and access control settings for pods           | `{}`                        |
| `securityContext`               | Privilege and access control settings for containers     | `{}`                        |
| `service.type`                  | Kubernetes service type                                  | `ClusterIP`                 |
| `service.port`                  | Kubernetes service port                                  | `80`                        |
| `podDisruptionBudget.enabled`   | Flag for enabling pod disruption budget                  | true                        |
| `podDisruptionBudget.rule.type` | Rule type of pdb                                         | `minAvailable`              |
| `podDisruptionBudget.rule.type` | Rule value of pdb                                        | `1`                         |
| `ingress.enabled`               | Flag for enabling ingress                                | false                       |
| `persistence.annotations`       | Kubernetes ingress annotations                           | `{}`                        |
| `ingress.labels`                | Ingress additional labels                                | `{}`                        |
| `ingress.hosts[0].host`         | Hostname to your Passbolt installation                   | `passbolt.organization.com` |
| `ingress.hosts[0].paths`        | Paths within the URL structure                           | `[]`                        |
| `ingress.tls`                   | Ingress secrets for TLS certificates                     | `[]`                        |
| `env`                           | Environment variables to attach to the pods              | `nil`                       |
| `secretsEnv`                    | Environment variables to attach to the pods from secrets | `nil`                       |
| `persistence.enabled`           | Flag for enabling persistent storage                     | false                       |
| `persistence.existingClaim`     | Do not create a new PVC but use this one                 | None                        |
| `persistence.storageClass`      | Storage class to be used                                 | ""                          |
| `persistence.accessMode`        | Volumes access mode to be set                            | `ReadWriteOnce`             |
| `persistence.size`              | Size of the volume                                       | 512Mi                       |
| `passbolt.baseUrl`              | Base URL where Passbolt will be accessible               | `passbolt.organization.com` |
| `passbolt.pro`                  | Flag for enabling pro license                            | false                       |
| `resources`                     | Passbolt Pod resource requests & limits                  | `{}`                        |
| `affinity`                      | Node / Pod affinities                                    | `{}`                        |
| `nodeSelector`                  | Node labels for pod assignment                           | `{}`                        |
| `tolerations`                   | List of node taints to tolerate                          | `[]`                        |

### Using external MySQL / MariaDB database

1. Create a yaml file `db-mysql.yaml` which will bind a local kubernetes host `db-mysql` to the external hostname of the database

```yaml
apiVersion: v1
kind: Service
metadata:
  name: db-mysql
spec:
  externalName: db.example.com
  type: ExternalName
```

2. Create a yaml file `db-passbolt.yaml` with a secret containing credentials to access database

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-passbolt
data:
  username: <Username in Base64>
  password: <Password in Base64>
```

3. Upload the two files

```shell
$ kubectl apply -f db-mysql.yaml
$ kubectl apply -f db-passbolt.yaml
```

4. Set the following values of the chart:

```yaml
env:
  DATASOURCES_DEFAULT_HOST: db-mysql
  DATASOURCES_DEFAULT_DATABASE: passbolt
secretsEnv: 
  - secretName: db-passbolt
  env:
    DATASOURCES_DEFAULT_USERNAME: username
    DATASOURCES_DEFAULT_PASSWORD: password
```

### Enabling pro license

1. Create a yaml file `passbolt-key.yaml` with a secret that contains one or more keys to represent the certificates that you want including

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: passbolt-key
data:
  subscription_key.txt: "<Content of subscription_key.txt in Base64>"
```

2. Upload your `passbolt-key.yaml` to a secret in the cluster you are installing Passbolt to.

```shell
$ kubectl apply -f passbolt-key.yaml
```

3. Set the following values of the chart:

```yaml
passbolt:
  pro: true
```
