# Kubernetes Intro "Lab" Exercise: Creating a K8s Cluster in AWS EKS

## June 6, 2021

## Goals

* Use AWS EKS in your personal AWS account to create an enterprise-ready k8s cluster with managed control plane in a repeatable way that we can easily "spin up" and then delete -- both today and for the k8s exercises we will do in the coming weeks.
* Explore the features of your k8s cluster using `kubectl` and the AWS web console to better understand how a k8s cluster is built and how it works.
* Deploy a simple web-server `deployment` & `service` in your cluster's default namespace.

## Prerequisites

1. If you haven't already, create a named IAM user in your AWS account with programmatic access and an `Administrator Access` IAM policy attached. You'll want to use this user to create you EKS cluster; it will need near-admin access, so creating an admin user is called for. Make sure you download the "Access key ID" and "Secret access key" credentials.
2. Make sure you have the AWS CLI installed (version 2: check via `aws --version`).
3. Install eksctl (see [doc here](https://docs.aws.amazon.com/eks/latest/userguide/eksctl.html)).
4. Install kubectl (see [doc here](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html).
   * Note: We will configure kubectl to access your k8s cluster *after* you create it using `eksctl`. If you have an existing `~/.kube/config` file, it will be automatically updated with an additional context for your new cluster. If you'd prefer not to update the`~/.kube/config` file you use for work, you can instead temporarily replace it via...
     * `mv ~/.kube/config ~/.kube/work_config`
     * ... (do the "lab" exercise; eventually delete your new k8s cluster)
     * `mv ~/.kube/config ~/.kube/learnanddevops_config`
     * `mv ~/.kube/work_config ~/.kube/config`

## Lab

1. (Ensure that you have "active" AWS credentials available in your terminal session for your named IAM admin user. If you try `aws sts get-caller-identity`, you should get a response that lists your active AWS CLI user; the ARN should match what you're expecting.)
2. Ensure that you have `kubectl` and `eksctl` installed:
   * Run `kubectl version` or `which kubectl`
   * Run `eksctl version` or `which eksctl`
3. Log in to your AWS account web console. Note the existing VPCs, running EC2s, Cloudformation stacks, and load balancers in your account before beginning the rest of the exercise. In addition to seeing a new cluster get created in the EKS console, you'll also see action in the VPC, EC2, and Cloudformation consoles once you start creating the cluster.
4. Use eksctl to create a k8s cluster named "learnanddevops" with managed control plane, 3 worker nodes, kubernetes version 1.19, in region "us-west-2" (the Oregon region):

       eksctl create cluster \
           --name learnanddevops \
           --version 1.19 \
           --region us-west-2 \
           --nodegroup-name worker-nodes \
           --node-type t3.micro \
           --nodes 3 --nodes-min 1 --nodes-max 4 \
           --managed

5. Open your Cloudformation web console and watch the cluster stack and the nodegroup stack finish creating. This will take 10-15 minutes. When it's done, check your EKS web console. You should see the new cluster listed there, along with its 3 active nodes and active control-plane containers listed in the cluster workloads. You'll also see additional resources have been created by Cloudformation (as instructed by `eksctl`, such as a VPC, NAT gateway, security groups, t3.micro EC2 instances for the worker nodes in the "worker-nodes" nodegroup, etc.). Look around a little and explore what got created.
6. If you're ready to create or update your kubectl config (see the Note on prerequisite 4), run

        aws eks update-kubeconfig --name learnanddevops --region us-west-2

7. Use `kubectl` commands to explore what's in your cluster, e.g. `kubectl get svc`. There won't be much to see yet, but you can practice a few kubectl `get` or `describe` commands and see what happens: [kubectl cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/#viewing-finding-resources).
8. Clone the following repo and change directories into the cloned project:

        git clone git@github.com:us-learn-and-devops/2021_05_26.git

9. You'll find two files there: `webapp-deployment.yaml` and `webapp-service.yaml`. Use the latter to create a load-balancer `service` object in your k8s cluster first:

        kubectl apply -f ./webapp-svc.yaml

10. Check your AWS web console for the new load balancer, and check kubectl to find the new `service` object.
11. Create the deployment:

        kubectl apply -f ./webapp-deployment.yaml

12. Use kubectl to check to make sure the deployment has been created. Try describing the deployment using kubectl. Use kubectl to list your pods in the default namespace and make sure you see the pods from the deployment. Finally, copy the load-balancer URL from either the EC2 web console or from the description of your `service` object in kubectl, and use a browser or `curl` to send a request to the web server running in your k8s deployment.
13. Make sure you can explain to yourself the path taken by the network request from your laptop to one of the 3 webapp pods running in your cluster.
14. When you're done playing with the deployment and with the cluster, use `eksctl` to clean up all the billable resources you created. (These will be easy to recreate again next time.)

        eksctl delete cluster learnanddevops

15. Check your AWS console. You should see that all the "stuff" that got created during the lab is now gone.
