
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
  <li>Go Inside <code>dev</code> directory</li>
  <li>Fill up the required variables in <code>dev.tfvars</code> like <code>profile</code> and <code>region</code></li>
  <li>Run <code>terraform init</code></li>
  <li>Run <code>terraform plan --var-file dev.tfvars</code> and see what are the resources to be deploying</li>
  <li>Run <code>terraform apply --var-file dev.tfvars</code> to deploy base setup like Vpc, Bastion, Ecr, and Eks-Cluster</li>
  <li> Export the kubeconfig <code>export KUBECONFIG=./kubeconfig_dev-eks-cluster</code>`
  <li>Update the kubeconfig of eks-cluster <code>aws eks update-kubeconfig --region [region-code] --name dev-eks-cluster</code> </li>
  <li> Run <code>terraform apply --var-file dev.tfvars</code> again to setup deployments inside eks-cluster and approve</li>
</ol>

#### CONFIGURE JENKINS PIPELINE ####
<ol>
  <li>Login to jenkins server</li>
  <li>Go inside <code>Manage jenkins/Manage Plugins/available</code> and install <code>nodejs</code> and <code>bitbucket</code> plugins</a></li>
  <li>Go inside <code>Manage jenkins/Global Tool Configuration/NodeJs</code>and follow:
    <ul>
       <li><code>Name: 14.16.0</code></li>
      <li><code>Version: NodeJS 14.16.0</code></li> 
      <li><code>Global npm packages to install: npm@6.14.12</code></li>
       <li>Save</li>
    </ul>
  </li>
  <li>
    Go inside `Manage jenkins/Credentials` Add credentials and follow:
    <ul>
      <li><code>Kind: Username with password</code></li>
      <li><code>Scope:ss-comprinno</code></li>
      <li><code>Password: `bitbucket-repository-password</code></li>
      <li><code>ID: BitBuketCredsForDockerfile</code></li>
      <li><code>Description: BitBuket Credentials for cloning repositories</code></li>
    </ul>
  </li>
  <li> On <code>dashboard/New Item/</code> and follow:
  <ul>
    <li><code>Item Name: dev-surveysparrow-pipeline</code></li>
    <li><code>Type: Pipeline</code></li>
    <li><code>Build Triggers: Build when a change is pushed to BitBucket</code></li>
    <li>Copy the dev-pipeline script in the terraform code repository and paste inside pipeline block</li>
    <li>Save</li>
  </ul> 
  </li>
  <li>Click on <code>Build Now</code> and check the pipeline is working</li>
  <li>Also create Webhook in bitbucket for continuous integration and continuous deployment</li>
  <li>Now commit a change in bitbucket repository and follow the pipeline till deployment completes</li>
</ol>
