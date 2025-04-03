# How to Install CloudDefense.AI DevSecOps on a Kubernetes cluster (AWS specific)
Note: Term 'cdefense' as an analogy for the on-prem installation

## STEP 1: PRE-REQUISITES (Customer Reponsibility)

Required Skills/Person:
1. **Mid to Senior DevOps resource/person with knowledge of Docker, Kubernetes, Helm, Cloud and your infrastructure.**
2. Time needed: Approx 2 hours for Pre-requisites, Approx 2 hours for CloudDefense.ai HELM chart.

## Pre-requisites for a cdefense installation on-premises

1. A Managed Postgres instance (for ex. AWS RDS db.r5.large) (Postgres is a Relational DB. Learn more about what is Postgres: https://www.postgresql.org/)
    1. enable automated backups for the RDS
2. A Kubernetes cluster (EKS/GKE/AKS) that has access to the above DB and to the internet with at least two nodegroups. (Learn more about what is Kubernetes: https://kubernetes.io/)
    1. node group for jobs
        1. each node has { label: job }
    2. node group for application pods
        1. (optional) each node has { label: cdefense }
    3. Access to the internet by installing Ingress controller. Doc: https://docs.aws.amazon.com/eks/latest/userguide/aws-load-balancer-controller.html 
3. A cluster auto-scaler should be installed for nodes auto scaling.
4. kubectl, helm packages needs to be installed.
5. Excel sheet with minimum infrastructure requirements: https://docs.google.com/spreadsheets/d/13R4DrVM6CfEgrlf3A7XDCrTNo8Aqq8DPU3Ne7FtHlgw/edit?usp=sharing

Confirming pre-requisites (How will you know that K8S is ready for helm installation): 
1. Here is a K8S command to check if your K8S cluster is up and running : `kubectl get nodes`
2. Here is a command to check if your K8S can access your Postgres DB : `pg_isready -d <db_name> -h <host_name> -p <port_number> -U <db_user>`
   - if postgres is not accessible check the rds security groups and the cluster security groups for the ports and any blocker from firewall
4. Here is a command to check if load balancer controller is installed in the EKS cluster: `kubectl get deployment -n kube-system aws-load-balancer-controller`
5. Here is a command to check if your K8S can connect to the internet : `kubectl get ingress` and check the address column in the output if the values are generated
6. Here is a command to check pod to pod communication inside K8S cluster: `kubectl exec -it <source-pod> -- ping <destination-pod-IP>`

## Resources that would be created
1. Kubernetes cluster(Instances as node)
2. EBS volumes for the nodes
3. RDS would be created and stroage(postgres)
4. Security groups for k8s and RDS
5. Application Load Balancer(Target groups, Listeners, Listeners policies, SSL Certificate) 
6. VPC(subnets, route tables, NAT Gateway, Elastic IP, Internet Gateway, db subnet group, Network ACL)
7. ACM
## Note 
- If Cluster auto scaler scales the node then extra instances and ebs volumes would be used.
- The ingress controller will create ALB and it's dependent services automatically(can't use exisitng load balancer) 

### Warnings

- Database URI has to be the Internal within the private network
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
