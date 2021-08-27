# About the project
A demo of workflow that integrates github actions, aws EKS and helm. The set up involves creating a  EKS cluster, building and pushing a new docker container image with tags to ECR upon code change using Github actions and finally - deploying the container images to EKS cluster using Helm charts   

# Built with
- awscli
- aws-iam-authenticator
- kubernetes-cli
- kubernetes-helm
- eksctl
- Git bash

# Getting Started

## Prerequisites
- Windows: chocolatey installer

```bash
choco install awscli aws-iam-authenticator kubernetes-cli kubernetes-helm eksctl
```
- Mac: homebrew installer 

```bash
brew install awscli aws-iam-authenticator kubernetes-cli kubernetes-helm  weaveworks/tap/eksctl
```
- Linux: apt-get (Ubuntu), yum (centos/rhel) installers or curl

```bash
For linux, install relevant apps using curl/yum/apt-get
```
- AWS account with a user having EKS IAM policy attached (https://docs.aws.amazon.com/eks/latest/userguide/service_IAM_role.html)

- Configure aws cli with access keys and default region
```bash
aws configure
```
## Installation

Using Git bash on desktop
#### Step 1: create a aws ECR repository to push application builds
```bash
aws ecr create-repository --repository-name kube-demo
```
 
#### Step 2: create a kube cluster with VPC, Subnet, Sec Group using eksctl.
eksctl is an official AWS tool to create VPC, Subnet, Sec Group, node groups and more required for a EKS cluster
```bash 
eksctl create cluster --name=kube-demo-cluster
```
This will take some time, meanwhile can move to other configurations
#### Step 3: Setup GitHub Repo and create Actions Workflow repo
- Fork the kube-demo repo into your GitHub user account -  https://github.com/sinhalvi/kube-demo
- Navigate to the .github directory in the GitHub web interface Code tab or native editor like VSCode. Click on the Actions tab and enable GitHub Actions workflow in this forked repo.
- View the workflow in your repo under .github/workflows/build.yml, ensure the region where EKS cluster resides. This should match the region in ~/.aws/config or the region specified on the eksctl command line.
```yaml
    AWS_DEFAULT_REGION: ${{ secrets.AWS_DEFAULT_REGION }}
    AWS_DEFAULT_OUTPUT: json
    AWS_ACCOUNT_ID: ${{ secrets.AWS_ACCOUNT_ID }}
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

#### Step 4: Configure secrets
In the build.yml file refer to secrets. Secrets are set up in the Github repo Settings tab, which are found at the bottom of the left navigation menu in the Secrets section.

Create four secrets called AWS_DEFAULT_REGION, AWS_ACCOUNT_ID, AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY with your AWS account ID (obtainable from the AWS console or by running aws sts get-caller-identity)and the new user’s key ID and the secret key (found in ~/.aws/credentials file). Secrets associated with your GitHub repo allow to keep sensitive data out of source code, but still make them available to GitHub Actions workflows.


#### Step 5: Configure nodejs app
Navigate to app/ folder which show the simple hello world nodejs pp. There is a dockerfile which is used convert this app into a docker image.

## App deployment workflow
1. Developers push changes to the app directory. 
2. GitHub Actions then builds and pushes container images to Elastic Container Repository (ECR), as specified by a workflow, .github/workflows/build.yml. 
3. Navigate to ECR to note the tag for the latest image.

## Using Helm chart to spin pods on EKS nodes
1. Navigate to kube-helm/values.yml. Change the value for containerimage with the latest docker image tag.
2. To create pods -
```bash 
helm install kube-demo-nodejs kube-helm/
```
To upgrade pods with new docker image
```bash 
helm upgrade kube-demo-nodejs kube-helm/
```
This would create the LoadBalancer service as well as pods in the eks nodes.

## Test Hello world
1. Execute
```bash 
kubectl get svc
```
to get the loadbalancer endpoint. 

2. hit the url http://<ELB-EndPoint> to check the end result.
```bash 
curl http://<ELB-EndPoint>
```

3. Notice the IP change with repeated hits which shows that both pods are active 

![alt text](https://github.com/sinhalvi/kube-demo/blob/main/test_result.png)
 
## Enhancement Ideas

#### Improvements (if had more time)
- SSL certificates and https for external facing urls
- Better versioning and tagging for docker image
- Wrapper script for entire installation and configurations
- Logging and Monitoring integration

### For better deployment process and ops
One of the two options below could be implemented to automate the application deployment process:

1. Using fluxcd:
Set up Flux on the cluster using [https://fluxcd.io/legacy/flux/get-started/]. Flux watches ECR for changes to the image listed in deployment configuration. When it detects a change, it updates the EKS cluster with the new image, no manual helm update needed

2. gitops deploy workflow
Setup another workflow using github actions which can detect the changes in helm chart and automatically apply helm upgrade to update pods with latest image

