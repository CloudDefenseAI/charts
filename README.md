# How to Install CloudDefense suite on a Kubernetes cluster

## Pre-requisites

There are three main pre-requisites for a production grade cdefense installation on-premises

1. A managed Postgres instance (for AWS RDS db.r5.large)
    1. enable automated backups
2. A kubernetes cluster (/examples/eks) with at least two nodegroups
    1. node group for jobs
        1. each node has { label: job }
    2. node group for all else
        1. (optional) each node has { label: cdefense }
3. A cluster auto-scaler

## WARNINGS

- Database URI has to be the Internal URI valid inside the private network
    - **DO NOT** obscure it behind a DNS as applications will be unable to connect to the database
- **DO NOT** change Database password or URI after helm install

## Install Helm Hooks



## Install cdefense

### Install cdefense from git repo

- clone the repo

    ```sh
    git clone https://github.com/CloudDefenseAI/charts
    ```

    ```sh
    cd charts
    ```
- create roles, role binding and service accounts

    ```sh
    kubectl apply -f charts/cdefense/rbac
    ```
- create secrets

    ```sh
    kubectl apply -f charts/cdefense/secrets
    ```
- create a dump folder (git ignored) if it does not exist

    ```
    mkdir dump
    ```
- create/edit values.yaml in dump (git ignored)

    ```sh
    cp charts/cdefense/values.yaml dump/cdefense/values.yaml
    ```
- Edit values.yaml in dump (git ignored) for ex. change domain and hostname

    ```sh
    vi dump/cdefense/values.yaml
    ```
- Install cdefense helm

    ```sh
    helm install cdefense charts/cdefense -f dump/cdefense/values.yaml --debug
    ```

### Install cdefense from helm repo

- add cdefense helm repo

    ```sh
    helm repo add cdefense https://clouddefenseai.github.io/charts/  
    ```
- update repos

    ```sh
    helm repo update
    ```
- create/edit values.yaml

    ```sh
    touch cdefense/values.yaml
    ```
- Edit values.yaml for ex. change domain and hostname

    ```sh
    vi cdefense/values.yaml
    ```
- Install cdefense

    ```sh
    helm install cdefense cdefense/cdefense -f cdefense/values.yaml --debug
    ```

## Install kafka

- Download the kafka helm repo (bitnami)

    ```sh
    helm repo add bitnami https://charts.bitnami.com/bitnami
    ```
- create/edit values.yaml

    ```sh
    touch kafka/values.yaml
    ```
- Edit values.yaml for ex. add a nodeSelector

    ```sh
    vi kafka/values.yaml
    ```

    ```yaml
    nodeSelector:
      label: external
    ```
- Install kafka helm

    ```sh
    helm install kafka bitnami/kafka -f kafka/values.yaml
    ```

## Configure Social Authentication

In order to sign in with different identity providers (for ex. github), create ID and secrets

### Github

- go to [github developer settings](https://github.com/settings/developers)
- create a New OAuth App
- Homepage URL is the base_url
- Authorization callback URL is <https://{base_url}/auth/realms/cdefense/broker/github/endpoint>

![](/images/github-auth.png)

### create secrets for authservice

- create a secret for authservice

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
      name: authservice-secrets
      type: Opaque
    stringData:
      SENDGRID_KEY: 
      GOOGLE_CLIENT_ID: 
      GOOGLE_CLIENT_SECRET: 
      GITHUB_CLIENT_ID: 
      GITHUB_CLIENT_SECRET: 
      GITLAB_APPLICATION_ID: 
      GITLAB_APPLICATION_SECRET: 
      BITBUCKET_KEY: 
      BITBUCKET_SECRET: 
      MICROSOFT_CLIENT_ID: 
      MICROSOFT_CLIENT_SECRET: 
    ```

    ```sh
    kubectl apply -f authservice-secrets.yaml
    ```

- restart authservice pod

## How to change location of logs

- update value.yaml

    ```yaml
    api:
      logs: 
        region: <REGION>
        bucket: <BUCKET>
    ```

### in case of private bucket

- Edit the scan-server-secrets.yaml file

  ```yaml
    AWS_SCAN_S3_ACCESS_KEY: <AWS_SCAN_S3_ACCESS_KEY>
    AWS_SCAN_S3_SECRET_KEY: <AWS_SCAN_S3_SECRET_KEY>
  ```

  ```sh
  kubectl apply -f scan-server-secrets.yaml
  ```

- or update secrets on cluster

    - encode values as base64 strings

    ```sh
    AWS_SCAN_S3_ACCESS_KEY=<AWS_ACCESS_KEY>
    BASE64_AWS_SCAN_S3_ACCESS_KEY=$(echo $AWS_SCAN_S3_ACCESS_KEY | base64)
    ```

    ```sh
    AWS_SCAN_S3_SECRET_KEY=<AWS_SECRET_KEY>
    BASE64_AWS_SCAN_S3_ACCESS_KEY=$(echo $AWS_SCAN_S3_SECRET_KEY | base64)
    ```

    - edit scan-server-secrets

    ```sh
    kubectl edit secret scan-server-secrets
    ```

    ```yaml
      AWS_SCAN_S3_ACCESS_KEY: <BASE64_AWS_SCAN_S3_ACCESS_KEY>
      AWS_SCAN_S3_SECRET_KEY: <BASE64_AWS_SCAN_S3_SECRET_KEY>
    ```

- save and restart api pod

  ```sh
  kubectl delete pod api-<some-string>
  ```
