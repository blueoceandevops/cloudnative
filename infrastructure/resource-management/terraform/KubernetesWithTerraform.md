# Installing Kubernetes on Oracle Cloud Infrastructure via Terraform


Welcome to our Kubernetes on Oracle Cloud Infrastructure (OCI) with Terraform series. This article walks you through the steps to deploy a Kubernetes cluster on OCI using Terraform with the OCI Terraform Provider. If you need assistance installing Terraform and/or the OCI Terraform Provider, please find the instructions to do so in our blog post: [Installing Terraform for Oracle Cloud Infrastructure](../installing-terraform-for-oracle-cloud-infrastructure/Readme.md)


### Install kubectl<br/>
Kubernetes is generally managed via a command line tool called "kubectl".

_Fun Fact:_ There is much debate over how this is pronounced, but "cube c t l" tends to be a safe bet. Other options include "cube ehctl", "cube control", and the ever-contentious "cube cuddle" (we do not recommend using this one in polite conversation). The name comes from the ability to control the Kubernetes cluster, thus, "kubectl".

* Mac:<br/>
  `brew install kubernetes`
* Windows - If you have chocolatey installed as described in _Installing Terraform for Oracle Cloud Infrastructure_, you can run:<br/>
  `choco install kubernetes-cli`
* Oracle Linux - If you have snap installed as described in _Installing Terraform for Oracle Cloud Infrastructure_, you can run: <br/>
  `sudo yum install kubectl --classic`


If none of these methods work for you or you just want to learn more about kubectl, you can find more [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/).


## Download the Terraform Kubernetes Installer
To begin, you will need to download the [Terraform Kubernetes Installer from GitHub](https://github.com/oracle/terraform-kubernetes-installer). This project consists of a Terraform template which will both create all of the infrastructure resources you need in Oracle Cloud Infrastructure, and install Kubernetes on those resources. There is a variety of ways to customize your Kubernetes deployment, which we will discuss more in the [Optional: Customize Your Kubernetes Terraform Deployment](#Optional:-Customize-Your-Kubernetes-Terraform-Deployment) section. You can also find more information about the Terraform Kubernetes Installer on GitHub.

### Mac
Download the Terraform Kubernetes Installer from GitHub.<br/>
`git clone https://github.com/oracle/terraform-kubernetes-installer.git
cd terraform-kubernetes-installer`

### Windows
1. Download the Terraform Kubernetes Installer from: https://github.com/oracle/terraform-kubernetes-installer<br/>
by clicking the "Clone or download" button and then clicking "Download ZIP".
2. Unzip the file.<br/>
`Expand-Archive -Path C:\Users\$env:UserName\Downloads\terraform-kubernetes-installer-master.zip -DestinationPath C:\Users\$env:UserName\Documents\TF_Windows_K8s`<br/>
3. Change to the new directory and run the rest of the commands in this guide from this directory.<br/>
`Set-Location C:\Users\$env:UserName\Documents\TF_Windows_K8s\terraform-kubernetes-installer-master`<br/>

### Oracle Linux
Download the Terraform Kubernetes Installer from GitHub.<br/>
`git clone https://github.com/oracle/terraform-kubernetes-installer.git
cd terraform-kubernetes-installer`

## Configuring Security Lists for Connection via kubectl

In order to connect to our Kubernetes deployment, we will need to add one or two (depending on the method you intend to use to access your applications running on Kubernetes) security list rules. We can do this using the _terraform.tfvars_ file.

The Terraform Kubernetes Installer comes with a file called terraform.example.tfvars which provides examples of some of the variables you can use to customize your Kubernetes deployment. The file can be found inside of the directory you downloaded the Terraform installer to (which you should now be in). Or you can see the current version of the file online at [GitHub](https://github.com/oracle/terraform-kubernetes-installer/blob/master/terraform.example.tfvars).

*TIP: You can also find some potentially useful variables in variables.tf, like region. Variables in this file can also be set via TF_VAR environment variables*

To use the _terraform.example.tfvars_ file to customize your deployment, first move or copy the file to a file named _terraform.tfvars_. In the _terraform.tfvars_ file, you can uncomment and modify the lines you wish to use. To enable connection from your local machine to the Kubernetes API Server, we will need to uncomment the following line in your _terraform.tfvars_ file:<br/>

`#master_https_ingress = "0.0.0.0/0"`
_Note that this option should not be enabled on production or other secure clusters._ This will tell Terraform to create a security list rule for the k8sMaster_security_list which will allow connections from ANYWHERE to port 443 on the Kubernetes Master nodes. To do this more securely, consider changing 0.0.0.0/0 to a narrower CIDR range which includes the machine from which you will be using kubectl.

There is another variable you can set in this file to easily configure access to applications running on Kubernetes via services of type NodePort. Configuring a service of type NodePort tells Kubernetes to expose port XX of your container on some port on the worker nodes of your cluster. If Kubernetes gave you port 30005 for your NodePort service for your application, then you can connect to that application by connecting to any worker IP at port 30005. It will look something like: worker_ip:30005. You can learn more about service types from our blog guide [Intro to Kubernetes Services](../../../distributed-systems-management/container-orchestration/kubernetes-101/service-types/Readme.md).

`#worker_nodeport_ingress = "0.0.0.0/0"`
This line will allow access to your applications exposed via NodePort from any external point. _Note the security risk in doing this._

`#worker_nodeport_ingress = "10.0.0.0/16"`
This line will allow access to your applications exposed via NodePort from any point WITHIN the OCI VCN - which the Kubernetes Terraform Installer creates by default with CIDR 10.0.0.0/16. This can be quite useful if you want other applications running in OCI to be able to interact with the applications running in Kubernetes.


## Optional: Customizing Your Kubernetes Terraform Deployment

If you like, you can also make other changes to the Kubernetes cluster deployment via this file. For example, if you wanted to use Bare Metal GPU instances for your Kubernetes Worker nodes, change the line in terraform.tfvars as follows:<br/>
`#k8sWorkerShape = "BM.Standard1.8"`<br/>
-><br/>
`k8sWorkerShape = "BM.GPU2.2"`

If you want to have one Kubernetes worker node deployed in each AD, uncomment these lines as shown below:<br/>
```
k8sWorkerAd1Count = "1"
k8sWorkerAd2Count = "1"
k8sWorkerAd3Count = "1"
```

If you're just trying things out, you can always go through with the default options and then make modifications later. This is also how to scale up or down the cluster with Terraform. We'll explore what scaling up or down using this same technique looks like after going through the steps to create the cluster.

## Creating a Kubernetes Cluster on OCI via the Kubernetes Terraform Installer
These commands will be the same on all operating systems. Make sure that you are running them in the directory of the Terraform Kubernetes installer which you downloaded in the last step.

1. To initialize the Terraform Kubernetes Installer, run:<br/>
`terraform init`<br/>
You should see a result similar to:<br/>

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.


2. To see what Terraform will create when it runs the Terraform Kubernetes Installer, run: <br/>
`terraform plan`

3. To create a Kubernetes cluster as described by the previous "plan" command, run: <br/>
`terraform apply`

## Connecting to Your Kubernetes Cluster
Once the terraform apply command has successfully completed, you can configure kubectl (Kubernetes' CLI-based tool for managing the cluster) on your local machine to manage the cluster in Oracle Cloud Infrastructure.
1. First, to be able to connect to your cluster from your local machine, you need a rule in the security list used for your Kubernetes master node(s). This rule should have been configured via the steps in [Configuring Security Lists for Connection via kubectl](#configuring-security-lists-for-connection-via-kubectl). If you did not perform this step, you can perform it now by making the change described in the section and rerunning `terraform plan` (which should show that you will make only one change) and then `terraform apply`.To remove this rule after confirming that you can connect to your cluster via kubectl, you can simply re-comment the line in _terraform.tfvars_ and re-run the Terraform commands to remove the rule.<br/>
2. Configure kubectl to connect to your new cluster using kubeconfig.
When Terraform created your cluster, it created a file called "kubeconfig" with the information required for kubectl to connect to your Kubernetes API server (including a token for security purposes). To configure kubectl to connect to the cluster you just created, we will use the Kubeconfig environment variable. On Mac or Linux machines, this looks like:<br/>
`export KUBECONFIG=`pwd`/generated/kubeconfig`<br/>
Once the Kubeconfig environment variable is set, you should be able to test that kubectl is connecting to your cluster by running:<br/>
`kubectl get nodes`<br/>
This should output something like:<br/>
NAME   STATUS    AGE       VERSION<br/>
k8s-master-ad1-0.k8smasterad1.k8sbmcs.oraclevcn.com  <br/> Ready     2m        v1.9.6<br/>
k8s-worker-ad1-0.k8sworkerad1.k8sbmcs.oraclevcn.com  <br/> Ready     34s       v1.9.6

_Note:_ Remember to remove the security list rule when you are done with this check if you do not want to leave the connection to your Kubernetes API Server open. Removing this rule will prevent you from using kubectl to manage this cluster, so you may want to consider simply modifying the variable in _terraform.tfvars_ to refer to a narrower CIDR range which includes the machine from which you are using kubectl.

## Scaling a Cluster Up or Down
You can use the terraform.tfvars (a copy or renamed version of the terraform.example.tfvars file discussed in the [Configuring Security Lists for Connection via kubectl](#configuring-security-lists-for-connection-via-kubectl) section) to scale the size of your cluster. If you make changes to the terraform.tfvars file after your cluster has been deployed and then rerun `terraform plan`, then Terraform will describe the changes the new terraform.tfvars file will make to your cluster. Running `terraform apply` will apply those changes. Let's walk through an example of scaling up the cluster you will create by default without making any modifications to terraform.example.tfvars.

The default cluster includes only one master and one worker node. You can see these nodes by running `kubectl get nodes`<br/>
![kubectl get nodes (default)](./images/kubectl_nodes_default.png "kubectl get nodes (default)")<br/>
In this output, you can see the current master and worker nodes.

In terraform.tfvars (created from terraform.example.tfvars) we add or change the following lines:<br/>
`k8sWorkerShape = "VM.Standard1.8"`<br/>
`k8sWorkerAd1Count = "1"`<br/>
`k8sWorkerAd2Count = "1"`<br/>
`k8sWorkerAd3Count = "1"`<br/>

Running `terraform plan` will output a detailed description of the changes along with a short line describing the changes like:
`Plan: 6 to add, 1 to change, 2 to destroy.`
This tells us that Terraform will destroy two objects, change one object, and add six objects with this change. This may seem like too many changes at first glance, but looking into the detailed output of the `terraform plan` shows that Terraform will also do work like modifying, adding, or removing security list rules.<br/>
*If you created a security list rule manually as described in this post, this change will remove that rule. To persist the rule allowing connectivity from the internet to your Kubernetes API Server, you will need to include that rule via terraform.*

If you run `terraform apply` with these changes, Terraform will take a couple of minutes to destroy the old worker(s) and create the new ones. Once the apply is complete, a `kubectl get nodes` command should output:<br/>
![kubectl get nodes (modified)](./images/kubectl_nodes_mod.png "kubectl get nodes (modified)")<br/>
In this output, you can see the same one master, as well as three new workers.

*NOTE: If you try to perform a scale up or scale down operation while running containers on your Kubernetes cluster, you will experience some brief downtime as Terraform performs destroys, and then creates.*

## Running a Smoke Test on Your Kubernetes Cluster
Once the terraform apply command has successfully completed, you should run a smoke test to confirm that your cluster is working properly.
We recommend following the smoke test outlined in our guide: [Performing a Smoke Test on a New Kubernetes Cluster](../../../distributed-systems-management/container-orchestration/kubernetes-101/smoke-tests/Readme.md)


