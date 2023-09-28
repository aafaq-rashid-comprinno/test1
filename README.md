# Comprinno Terraform Code for EKS-Boilerplate
​
## About Terraform

**Terraform** is a tool for building, changing, and versioning infrastructure safely and efficiently. Terraform can manage existing and popular service providers as well as custom in-house solutions.

## Technical details

All resources are deployed on AWS and use Terraform for Infrastructure as code.

## Steps to Update Terraform providers

```bash
terraform state replace-provider registry.terraform.io/-/template  registry.terraform.io/hashicorp/template
terraform state replace-provider registry.terraform.io/-/aws  registry.terraform.io/hashicorp/aws
terraform state replace-provider registry.terraform.io/-/archive  registry.terraform.io/hashicorp/archive
```
​
## Prerequisites:

- You need to create an s3 bucket and dynamoDB table beforehand to store the `.tfstate` for Terraform backend mentioned in the `main.tf` files.

- **You must have Terraform installed**

    1. To check if it's installed, run:

        ```bash
        terraform version
        ```

    2. If Terraform is not installed or to upgrade to the latest version, refer [this doc](https://learn.hashicorp.com/tutorials/terraform/install-cli)

    **Note**: Make sure Terraform version should be `0.15.0` or above, as this package is configured with this very version.

- **You must have AWS CLI installed**

    1. To check if it's installed, run:

        ```bash
        aws --version
        ```

    2. If AWS CLI is not installed or to upgrade the AWS CLI version, refer [this doc](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

    3. Configure AWS CLI using the Access-Secret Keys of the IAM user with the following command:

        ```bash
        aws configure
        ```

        **Note**: Make sure to have AWS CLI version `2.x` installed.

- **You must have kubectl(>= v1.23) installed**

    1. To check if it's installed, run:

        ```bash
        kubectl version --client
        ```

    2. If kubectl is not installed or to upgrade the version, refer [this doc](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

- **Connection for the CodePipeline must be created beforehand**
- **Update the argocd-ssh-key and Netrc parameter values in aws parameter store post infrastructure creation using terraform**
- **Add the microservice configuration in S3 bucket within respective microservice folder names**

## What this code creates?

**Note: VPC creation is optional and is controlled by a boolean variable.**

* `VPC and related resources` 
   - VPC
   - NAT
   - Internet gateway
   - Subnets 
   - Route tables
   - Routes
   
* `AWS Elastic Kubernetes Service Cluster`
   - EKS Cluster with enabled IRSA(IAM Roles for Service Accounts)
   - EKS Cluster role
   - AWS launch template
   - Nodes
   - Node IAM role
   - Auto-scaling group
   - Elastic block storage
   - OIDC identity provider 
   - Security Group
   - AWS EFS 
   - KMS CMK Keys
   
* `Kubernetes resources` (In a separate terraform apply)
    - AWS Load Balancer Controller and its dependencies
    - Load balancer controller Role
    - Cluster Autoscaler and its dependencies
    - Cluster auto scaler role
    - Metrics Servers
    - Prometheus (using helm)
    - Grafana (using helm)
    - EBS CSI controller
    - EFS CSI controller
    - Service Account
    - ArgoCD
  
  
* `CICD (sample nginx application)` (as part of the base infra)
    - AWS S3 bucket for CodePipeline
    - AWS S3 bucket for microservice configurations
    - AWS CodePipeline
    - AWS CodeBuild project
    - AWS IAM policy and role for CodeBuild and CodePipeline
​
**Note:** Creation of the following resources in the `example.tfvars` file are optional.
- Creation of the following resources in `aws_base_infra` are controlled by a boolean flag. To create these resources, their respective flags must be set to `true`, else `false`.

    * VPC
    * EFS
    * ECR

- Similarly, creation of the following resources in `kubernetes_service_infra` are controlled by a boolean flag. To create these resources, their respective flags must be set to `true`, else `false`.

    * Autoscaler
    * AWS LoadBalancer Controller
    * Metrics Server
    * Grafana
    * Prometheus
    * EBS CSI
    * EFS CSI
    * Service Account
    * ArgoCD

## Attaching Policies to the Node Role

* Node role can be accessed from Amazon EKS console > Click on the cluster > compute tab > click on the node group. In the details tab, the node role arn and link are present. Click on the link which will direct you to the node role in IAM console. Attach the above-mentioned policies to the node role.

## Access Required

**1**. To apply and create AWS EKS cluster using this code, one needs to have Administrator access to AWS account.

**2**. To give access to the cluster to other users, you need to add IAM user to the configmap `aws-auth.yml` of the cluster. The sample `aws-auth.yml` is given with this code which can be edited and then applied to the cluster using the below command.

    kubectl apply -f aws-auth.yml -n kube-system

## Important Notes:

* AWS EKS must be launched in private subnets.

* Route53/DNS updates will not be part of this code.

* Route53/DNS update entries are to be made for the load balancer.

* Pushing Microservices images to ECR will not be part of this code.

* All security resources enabling (SecurityHub, Config, Guardduty) will not be part of the code.

* In distributed terraform apply operations, if there are any terraform null-resource with kubectl commands that are to be run, make sure you have `kubeconfig` file present in the root folder. If not, you can download the `kubeconfig` file with this command in the root folder:

        aws eks update-kubeconfig --name <cluster-name> --region <eks-region>

* Also, for accessing the UI of Grafana deployed on Kubernetes, the links and credentials are shared separately.

## Assumptions

* You want to create an EKS cluster with an autoscaling group of `Managed` Nodes, their corresponding dependencies, Kubernetes resources, and other tools/services on Kubernetes.

* You have a Linux system. For Windows users, as the EKS module used in the code may give issues, please read the following [doc](https://github.com/terraform-aws-modules/terraform-aws-eks/blob/master/docs/faq.md#deploying-from-windows-binsh-file-does-not-exist).

## Walkthrough of the Folder Structure

The folder structure of the complete package will be as follows:

- aws_base_infra
- kubernetes_service_infra
- modules
- example.tfvars
- README.md


## Deploying the AWS Base Infra

- Once you have the code, change directory to `comprinno-eks-boilerplate` folder. 

- Create a new `.tfvars` file with reference from `example.tfvars`, and update all the required flag values according to the use in the new file. 

- Install the necessary binaries required for the resource creations of `aws_base_infra`, run:

        cd aws_base_infra

        terraform init

- Once all the binaries are installed in the same directory, run:

        terraform apply --var-file="<path to the new tfvars file>/<.tfvars file name>"


## Deploying the Kubernetes Service Infra

- After provisioning aws base infra resource change directory to `comprinno-eks-boilerplate` folder.

- Install the necessary binaries required for the resource creations of `kubernetes_service_infra`, run:

        cd kubernetes_service_infra

        terraform init

- To create Kubernetes-related resources, run:

        terraform apply --var-file="<path to the new tfvars file>/<.tfvars file name>"

## Destroying the Resources

- To destroy all the resources, change directory to `kubernetes_service_infra`, run:

        terraform destroy --var-file="<path to the new tfvars file>/<.tfvars file name>"

- To destroy all AWS resources, change directory to `aws_base_infra` folder and run:

        terraform destroy --var-file="<path to the new tfvar file>/<.tfvars file name>"

**NOTE: To destroy all the resources, you have to destroy Kubernetes resources first and then the base infra.**

***NOTE:*** *To run a module separately, use the following command:*

    terraform apply -target=module.<module_name> --var-file="<full path of tfvars file Name>/<.tfvars file name>"
