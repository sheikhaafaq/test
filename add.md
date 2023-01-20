# Comprinno Terraform Code for EKS-Boilrplate

## **About Terraform**

**Terraform** is a tool for building, changing, and provisioning infrastructure safely and efficiently. Terraform can manage various service providers as well as custom in-house solutions.


## Technical details
All resources are deployed on AWS and uses Terraform for Infrastructure as a code (IaaC)


## Steps to Update Terraform providers

terraform state replace-provider registry.terraform.io/-/template  registry.terraform.io/hashicorp/template
terraform state replace-provider registry.terraform.io/-/aws  registry.terraform.io/hashicorp/aws
terraform state replace-provider registry.terraform.io/-/archive  registry.terraform.io/hashicorp/archive

## Prerequisites:

* Create a s3 bucket and dynamoDB table beforehand to store the terraform's .tfstate backend mentioned in the main.tf files


* You should have Terraform installed

    > To check if it's installed, run:

        terraform version

    > If Terraform is not installed or to upgrade to latest version, refer [this doc](https://learn.hashicorp.com/tutorials/terraform/install-cli)

    > Make sure terraform version sould be 0.15.0 or above, as this package is configured with this very version.

*  You have AWS CLI installed

    > To check if it's installed, run:

        aws --version

    > If AWS CLI is not installed or to upgrade to version aws cli version 2, refer [this doc](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).
    
    > Configure AWS CLI to use Access Keys of IAM user with following command:

        aws configure

    > Make sure the aws cli version 2.x is installed

* You have docker installed. 
    > To check if it's installed, run:

        docker version

    > If docker is not installed or to upgrade to latest version, refer [this doc](https://docs.docker.com/engine/install/).

    > Make sure major docker version 20.x is installed

* You have kubectl(>= v1.23) installed. 
    > To check if it's installed, run:

        kubectl version --client

    > If kubectl is not installed or to upgrade to latest version, refer [this doc](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

* You have AWS  IAM Authenticator installed. 
    > To check if it's installed, run:

        aws-iam-authenticator help

    > If IAM Authenticator is not installed or to upgrade to latest version, refer [this doc](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html).

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
   - Auto scaling group
   - Elastic block storage
   - OIDC identity provider 
   - Security group
   - AWS EFS 
   - KMS CMK Keys
   
* `Kubernetes resources` (In a separate terraform apply)
    - AWS Load Balancer Controller and its dependencies
    - Load balancer controller Role
    - EFS CSI controller
    - Cluster Autoscaler and its dependencies
    - Cluster auto scaler role
    - Metrics Servers
    - Fluentbit and its dependencies
    - Prometheus (using helm)
    - Grafana (using helm)
    - EBS CSI controller
    - Calico
    - Ingress resources
  
**Note: Creation of the following resources in the aws-base-infra.tfvars are optional. To create these resources, thier respective flags must be set to true else false.**
* 
    * vpc             
    * nat             
    * efs            
    * public_subnet   
    * eks_control_plane_subnet 
    * eks_data_plane_subnet 
    * db_subnet       
    * public_route_table 
    * eks_route_table 
    * db_route_table  
    * ebs 
    * kms                   
    * security_groups       
    * security_groups_rules 
    * route_table_A  
       


## Manual Steps

* In order to spin-up the resources in existing VPC, to make the data call based on tag or tag-set, please add additional tags for public and private subnets 
:
	for public subnets add "kubernetes.io/role/elb" = "1"
	for private subnets add "kubernetes.io/role/internal-elb" = "1"

## Access Required

* To apply and create AWS EKS cluster using this code one needs to have Administrator access of AWS account.

* To give access of the cluster to other users, you need to add IAM user to the configmap `aws-auth.yml` of the cluster. The sample aws-auth.yml is given with this code which can be edited and then applied to the cluster using the below command.

    `kubectl apply -f aws-auth.yml -n kube-system` 

## Important Notes:

* AWS EKS must be launched in private subnets.

* Route53 / DNS updates will not be part of this code.

* Route 53 / DNS updates entries are to be made for the loadbalancer.

* Pushing Microservices images to ECR will not be part of this code.

* All security resources enabling (SecurityHub, Config, Guardduty) will not be part of the code

* In distributed terraform apply operations, if there are any terraform null-resource with kubectl commands that are to be run, make sure you have kubeconfig file present in the root folder. If not, you can download the kubeconfig file with this command in root folder:

      aws eks update-kubeconfig --kubeconfig kubeconfig_<your cluster name> --name <your cluster name> --region <your deployment region code>


* Also for accessing the UI of prometheus and grafana deployed on kubernetes, the links and credentials are shared seperately.

## Assumptions

* You want to create an EKS cluster with autoscaling group of `Managed` Nodes, its corresponding dependencies, Kubernetes resources and other tools/services on kubernetes.
* You have a Linux system. For windows users, as EKS module used in the code may give issues, please read the following [doc](https://github.com/terraform-aws-modules/terraform-aws-eks/blob/master/docs/faq.md#deploying-from-windows-binsh-file-does-not-exist).
 

## Deploying Terraform code

Once you have the code, goto the `bb-eks` folder and execute the following commands.

> Note: Make sure you have terraform installed in your system. Check the instructions [here](../readme.md) if you have setup terraform.

- The file structure is split into 3:
    * aws-base-infra: Refers the base infrastructure resources creation.
    * kubernetes-ervice-infra: Refers the kubernetes, associated observability and other resources creation.
    * modules: Contains the code to spin-up the base-infra and kubernetes resources applies from above 2 files.

- The .tfvar files (which feeds the necessary variables into resources) are also devided into two based on the file structure, namely: 
    * aws-base-infra.tfvars and 
    * kubernetes-base-infra.tfvars, for aws-base-infra and kubernetes-service-infra with all the depending values respectively.
    
- Create new var-files with reference from example.tfvars, and update all the required values in that file.

- While creating .tfvar file please consider choosing the correct type & size of EC2 intance in the eks_conf:node_group ection to successfully create all the pods and to avoid certain pods going in the pending state

- Make sure the aws configure should be in region where we want to deploy the resources. To make a region default, use the command:

        aws configure set --region <region-name>

- To install necessary binaries required for the resource creations, from this root directory, run:
    
        terraform init

- To create base infrastructure resources, go to aws-base-infra folder and run:

        terraform apply --var-file="<new-var-file-name>"

# Note: By default the base-infra will create:
    * A new VPC.
    * Creates an internet gateway.
    * 3 subnets for each web(public), app(private), and db(private) and map them in 3 different AZs of a    chosen region.
    * Creates the corresponding reouting tables and routes and associates the subnets.
    * Creates a NAT gateway with an elastic IP address to connect app subnets to the internet.

- To create resources in existing VPC with various sort of base infra customization, resource flags are provided to control the base infra resources creation. The aws-base-infra.tfvars' flags are as following:
    * vpc             
    * nat             
    * efs             
    * public_subnet   
    * eks_control_plane_subnet 
    * eks_data_plane_subnet 
    * db_subnet       
    * public_route_table 
    * eks_route_table 
    * db_route_table  
    * ebs 
    * kms                   
    * security_groups       
    * security_groups_rules 
    * route_table_A         

    These flags accept one of the two values true or false. With true, the resource will be created and with false the existing resource will be selected.

# Note: For existing vpc details, a module named as vpc-existing has been added in aws-base-infra's main.tf which does the data call based on name tag to get the following resources id's and uses them wherever required:

    * vpc_name = <name tag of existing VPC>
    * public_subnets_names = [<list of name tags of public subnets in existing VPC>]
    * control_plane_eks_subnet_names = [<list of name tags of private control plane subnets in existing VPC>]
    * data_plane_eks_subnet_names = [<list of name tags of private data plane subnets in existing VPC>]
    * db_subnets_names = [<list of name tags of private db subnets in existing VPC>]

# Note: Since the default security groups are included with the cluster package, to add custom security groups, a different terraform apply is required to make changes in place.

- Once the base-infra creates, it generates a kube config file with an alpha version of k8s API, which must be replaced to beta verion. To replace it to the beta version, use the following command:

        aws eks update-kubeconfig --name <cluster-name> --region <eks-region>


- To create kubernetes resources, go to the kubernetes-service-infra folder and run:
        
        terraform init
        
        terraform apply --var-file="../<new-var-file-name>"



- To destroy all Kubernetes resources, first go to kubernetes-service-infra directory and run:

        terraform destroy --var-file="../<var-file-name>"
        
- To destroy all AWS resources, go to the aws-base-infra folder and run:

        cd ../
        terraform destroy --var-file="<var-file-name>"
        
        
**NOTE: To destroy all the resource, you have to destroy kubernetes resources first and then the base infra.**
