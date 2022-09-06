# How to Install CloudDefense suite on a Kubernetes cluster

## Install kafka

- Download the kafka helm repo (bitnami)

    ```
    helm repo add bitnami https://charts.bitnami.com/bitnami
    ```

- Install kafka helm

    ```
    helm install kafka bitnami/kafka
    ```

## Install cdefense

- add cdefense helm repo

    ```
    helm repo add cdefense https://clouddefenseai.github.io/charts/  
    ```
- update repos

    ```
    helm repo update
    ```
- clone the repo
- create roles, role binding and service accounts

    ```
    kubectl apply -f charts/cdefense/rbac
    ```
- create secrets

    ```
    kubectl apply -f charts/cdefense/secrets
    ```
- Install cdefense helm

    ```
    helm install cdefense cdefense --debug
    ```

    or

    ```
    helm upgrade cdefense cdefense/cdefense --debug
    ```

## Configure Credentials

In order to sign in with different identity providers (for ex. github), create ID and secrets 

## Github

- go to [github developer settings](https://github.com/settings/developers)
- create a New OAuth App
- Homepage URL is the base_url
- Authorization callback URL is https://{base_url}/auth/realms/cdefense/broker/github/endpoint

![](/images/github-auth.png)

## create secrets on kubernetes cluster

- create a secret for authservice or use a yaml file


```
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

```
kubectl apply -f authservice-secrets.yaml
```
- restart authservice pod
