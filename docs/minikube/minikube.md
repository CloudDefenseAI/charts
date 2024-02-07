# Minikube

## Pre-requisites

- [Docker](https://www.docker.com/)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)
- [Minikube](https://minikube.sigs.k8s.io/)
- [ngrok](https://ngrok.com)

## Set up Minikube Cluster

- start local kubernetes cluster

  ```
  minikube start --memory 8096 --cpus 2 --profile cdefense
  ```
- enable nginx addon

  ```
  minikube addons enable ingress --profile cdefense
  ```
- (optional) wait for pod to reach running state

  ```
  kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
  ```
- get minikube IP (for ex. 192.168.49.2)

  ```
  IP_ADDR=$(minikube ip -p cdefense)
  echo $IP_ADDR
  ```

## Install Kafka

- Download the kafka helm repo (bitnami)

  ```sh
  helm repo add bitnami https://charts.bitnami.com/bitnami
  ```
- (optional) create/edit `values.yaml`

  ```
  nodeSelector:
    label: external
  ```
- Install kafka helm

  ```sh
  helm install kafka bitnami/kafka -f values.yaml
  ```

## Install cdefense

- add cdefense helm repo

    ```sh
    helm repo add cdefense https://clouddefenseai.github.io/charts/  
    ```

- update repos

    ```sh
    helm repo update
    ```

- clone the repo

    ```
    git clone https://github.com/CloudDefenseAI/charts
    ```
- create roles, role binding and service accounts

    ```sh
    kubectl apply -f charts/cdefense/rbac
    ```

- create secrets

    ```sh
    kubectl apply -f charts/cdefense/secrets
    ```

- Install cdefense helm

    ```sh
    helm install cdefense cdefense --debug
    ```

    or

    ```sh
    helm upgrade cdefense cdefense/cdefense --debug
    ```

## How to run scans against cdefense running on a Local kubernetes cluster

- create/edit `./secrets/ngrok-secrets.yaml`
- create ngrok-secrets

  ```
  kubectl apply -f ./secrets/ngrok-secrets.yaml
  ```

- scans requires a public external url because request to the scan-api endpoint is sent from inside the container
- get the port ngrok dashboard

    ```
    NGROK_PORT=$(kubectl get svc ngrok-service -o=jsonpath='{.spec.ports[?(@.port==4040)].nodePort}')
    ```
- go to ngrok dashboard i.e. ${IP_ADDR}:{NGROK_PORT}/status and copy the PUBLIC_URL address to scan server for ex. http://c6fe-49-207-223-136.ngrok.io

  ```
  export NGROK_URL=c6fe-49-207-223-136.ngrok.io"
  ```
- substitute PUBLIC_URL to get correct SCAN_URL inside the command to run a scan or job (MAKE SURE api_key is correct)

    ```sh
    docker run -it -e "CONTAINER_SCAN_URL=http://${ NGROK_URL }/scan/container" --rm cdefense/container-scan:latest --image_name=vulnerables/cve-2014-6271 --app_name=test --api_key=<API_KEY>
    ```

    or

    ```yaml
    apiVersion: batch/v1
    kind: Job
    metadata:
    name: java-sca-job
    spec:
    template:
        spec:
        containers:
        - name: java-job
            image: cdefense/reposcan:java
            command: ["reposcan.sh", "sca",  "--project-name", "test", "--api-key", ${API
            _KEY}]
            env:
            - name: GIT_REPO
            value: "https://github.com/cdefense/vulnerable-java-maven.git"
            - name: SCAN_URL
            value: http://${ NGROK_URL }
        restartPolicy: Never
    backoffLimit: 1
    ```
