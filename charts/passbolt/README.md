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

After startup of the main pod, you need to manually create the first administrator with the following command inside the container :

```bash
bin/cake passbolt register_user -u <email> -f <firstname> -l <lastname> -r admin
```

Then, follow the instructions printed on screen.

## Uninstalling the chart

To uninstall/delete the deployment:

```bash
$ helm list
NAME           REVISION    UPDATED                     STATUS      CHART              NAMESPACE
kindly-newt    1           Mon Oct  5 11:08:43 2020    DEPLOYED    passbolt-1.0.1    default
$ helm delete kindly-newt
```

## Configuration

The following table lists the configurable parameters of the Passbolt chart and their default values.

| Parameter                       | Description                                              | Default                     |
| ------------------------------- | -------------------------------------------------------- | --------------------------- |
| `image.repository`              | image repository                                         | `passbolt`                  |
| `image.tag`                     | `passbolt` image tag.                                    | `2.13.5-debian`             |
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
| `ingress.className`             | Ingress className for multi ingress env                  | `nginx`                     |
| `email.host`                    | Email server host name                                   | `smtp.example.com`          |
| `email.port`                    | Email server port number                                 | `25`                        |
| `email.from`                    | Email from field                                         | `noreply@example.com`       |
| `email.username`                | Username for login to email server                       | `nil`                       |
| `email.password`                | Password for login to email server                       | `nil`                       |
| `email.timeout`                 | Email default connection timeout                         | `30`                        |
| `env`                           | Environment variables to attach to the pods              | `nil`                       |
| `secretsEnv`                    | Environment variables to attach to the pods from secrets | `nil`                       |
| `persistence.enabled`           | Flag for enabling persistent storage                     | false                       |
| `persistence.existingClaim`     | Do not create a new PVC but use this one                 | None                        |
| `persistence.storageClass`      | Storage class to be used                                 | ""                          |
| `persistence.accessMode`        | Volumes access mode to be set                            | `ReadWriteOnce`             |
| `persistence.size`              | Size of the volume                                       | 512Mi                       |
| `db.host`                       | Hostname of MariaDB                                      | `passbolt-mariadb`          |
| `db.name`                       | Database name used by Passbolt                           | `passbolt`                  |
| `mariadb.enabled`               | Flag for provisioning an embedded MariaDB instance       | true                        |
| `mariadb.fullnameOverride`      | Service name for accessing embedded MariaDB instance     | `passbolt-mariadb`          |
| `mariadb.auth.database`         | Default name for database to create                      | `passbolt`                  |
| `mariadb.auth.username`         | Default username for the database to create              | `passbolt`                  |
| `mariadb.auth.password`         | Default password for the database to create              | `password`                  |
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
db:
  host: db-mysql
  name: passbolt
  existingSecret:
    name: db-passbolt
    usernameKey: username
    passwordKey: password

mariadb:
  enabled: false
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
  license:
    enabled: true
    existingSecret:
      name: passbolt-key
      licenseKey: subscription_key.txt
```
