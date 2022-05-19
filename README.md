
# SURVEYSPARROW DOCUMENTATION #

## TERRAFORM CODE IMPLEMENTATION #

### PREREQUISITES ###
#### Install tools in you local system ####
<ul>
  <li><a href="https://kubernetes.io/docs/tasks/tools/">Kubectl</a></li>
  <li><a href="https://helm.sh/docs/intro/install/">Helm</a></li>
  <li><a href="https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html">Aws-Iam-Authenticator</a></li> 
</ul>


#### STEPS TO FOLLOW ###
<ol>
  <li>Configure credentials for awscli `aws configure`</li>
  <li><a href="https://kubernetes.io/docs/tasks/tools/">Clone the terraform code repository</a></li>
  <li>Go Inside `dev` directory</li>
  <li>Fill up the required variables in `dev.tfvars`like `profile` and `region`</li>
  <li>Run `terraform init`</li>
  <li>Run `terraform plan --var-file dev.tfvars` and see what are the resources to be deploying</li>
  <li>Run `terraform apply --var-file dev.tfvars` to deploy base setup like Vpc, Bastion, Ecr, and Eks-Cluster</li>
  <li> Export the kubeconfig `export KUBECONFIG=./kubeconfig_dev-eks-cluster`
  <li>Update the kubeconfig of eks-cluster `aws eks update-kubeconfig --region [region-code] --name dev-eks-cluster </li>
  <li> Run `terraform apply --var-file dev.tfvars` again to setup deployments inside eks-cluster and approve</li>
</ol>

#### CONFIGURE JENKINS PIPELINE ####
<ol>
  <li>Login to jenkins server</li>
  <li>Go inside `Manage jenkins/Manage Plugins/available` and install `nodejs` and `bitbucket` plugins</a></li>
  <li>Go inside `Manage jenkins/Global Tool Configuration/NodeJs`and follow:
    <ul>
       <li>Name: 14.16.0</li>
       <li>Version: NodeJS 14.16.0</li> 
       <li>Global npm packages to install: npm@6.14.12</li>
       <li>Save</li>
    </ul>
  </li>
  <li>
    Go inside `Manage jenkins/Credentials` Add credentials and follow:
    <ul>
        <li>Kind: Username with password</li>
        <li>Scope:ss-comprinno</li>
        <li>Password: `bitbucket-repository-password`
        <li>ID: BitBuketCredsForDockerfile</li>
        <li>Description: BitBuket Credentials for cloning repositories</li>
    </ul>
  </li>
  <li> On dashboard/New Item/ and follow:
  <ul>
    <li>Item Name: dev-surveysparrow-pipeline</li>
    <li>Type: Pipeline</li>
    <li>Build Triggers: Build when a change is pushed to BitBucket</li>
    <li>Copy the dev-pipeline script in the terraform code repository and paste inside pipeline block</li>
    <li>Save</li>
  </ul> 
  </li>
  <li>Click on `Build Now` and check the pipeline is working</li>
  <li>Also create Webhook in bitbucket for continuous integration and continuous deployment</li>
</ol>
