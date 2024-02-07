# Setup tools required to create cluster and deploy 

## Pre-requisities

AWS EC2 instance ubuntu 20.04

## Configure security group of the EC2 Instance

In security group, open the port 22(remove other ports)

![alt text](images/Screenshot 2024-02-07 at 7.29.48 PM.png)

## SSH to Bastion-Host EC2 instance

### Install awscli on EC2 instance

```sh
sudo apt update
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
```
```sh
sudo apt-get install unzip
```
```sh
unzip awscliv2.zip
```
```sh
sudo ./aws/install
```

### Install kubectl on EC2 instance

```sh
curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.19.6/2021-01-05/bin/linux/amd64/kubectl
```
```sh
chmod +x ./kubectl
```
```sh
sudo mv ./kubectl /usr/local/bin
```
```sh
kubectl version --short --client
```

### Install eksctl on EC2 instance

```sh
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
```
```sh
sudo mv /tmp/eksctl /usr/local/bin
```
```sh
eksctl version
```

### Install helm on EC2 instance

```sh
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

### Install postgresql-clinet on EC2 instance

```sh
sudo apt-get install -y postgresql-client
```
```sh
psql --version 
```
