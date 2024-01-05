# How to Install CloudDefense.AI DevSecOps on a Kubernetes cluster
Note: Term 'cdefense' as an analogy for the on-prem installation

## STEP 1: PRE-REQUISITES (Customer Reponsibility)

Required Skills/Person:
1. **Mid to Senior DevOps resource/person with knowledge of Docker, Kubernetes, Helm, Cloud and your infrastructure.**
2. Time needed: Approx 2 hours for Pre-requisites, Approx 2 hours for CloudDefense.ai HELM chart. May also require a video call for debugging session

There are three main pre-requisites for a cdefense installation on-premises

1. A managed Postgres instance (for ex. AWS RDS db.r5.large) (Postgres is a Relational DB. Learn more about what is Postgres: https://www.postgresql.org/)
    1. enable automated backups
2. A Kubernetes cluster (EKS/GKE/AKS) that has access to the above DB and to the internet (/examples/eks) with at least two nodegroups. (Learn more about what is Kubernetes: https://kubernetes.io/)
    1. node group for jobs
        1. each node has { label: job }
    2. node group for all else
        1. (optional) each node has { label: cdefense }
    3. Access to the internet by installing Ingress controller. Doc: https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html 
3. A cluster auto-scaler

Excel sheet with minimum infrastructure requirements: https://docs.google.com/spreadsheets/d/13R4DrVM6CfEgrlf3A7XDCrTNo8Aqq8DPU3Ne7FtHlgw/edit?usp=sharing

Confirming pre-requisites (How will you know that K8S is installed?): 
    1. Here is a K8S command to check if your K8S cluster is up and running : `kubectl get nodes`
    2. Here is a command to check if your K8S can access your Postgres DB : `...` 
    3. Here is a command to check if your K8S can connect to the internet : `...` 

### WARNINGS & DEBUGGING

- Database URI has to be the Internal URI valid inside the private network
    - **DO NOT** obscure it behind a DNS as applications will be unable to connect to the database
- **DO NOT** change Database password or URI after helm install
- In case of firewall blocking, you may need to whitelist urls: https://storage.googleapis.com/, https://gcr.io/v2/, https://registry.k8s.io/v2/, https://index.docker.io/v2/, https://github.com/CloudDefenseAI/charts, IP address of the NAT gateway


## STEP 2: INSTALL CLOUDDEFENSE.AI (CloudDefense.ai on-prem installation)
Note: Term 'cdefense' as an analogy for the on-prem installation


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
