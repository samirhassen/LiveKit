
# Requirements 
- AWS cli with credentials eksctl
- kubernetes-helm
- Aws cli, you can install it from npm package as well, nodejs, aws-cdk package from npm

Open terminal, configure aws cli to access your account run > aws configure

Run bootstraping cdk bootstrap aws://ACCOUNT-NUMBER/REGION

Create the cluster > eksctl create cluster --name my-cluster --region region-code

Add IAM rules for the cluster, for more detailed instruction: https://docs.aws.amazon.com/eks/latest/userguide/lbc-helm.html

# Create IAM policy
First download the policy config file run > curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.7.2/docs/install/iam_policy.json

Run > aws iam create-policy \ --policy-name AWSLoadBalancerControllerIAMPolicy \ --policy-document file://iam_policy.json

# Create a service account:

Run > eksctl create iamserviceaccount \ --cluster=my-cluster \ --namespace=kube-system \ --name=aws-load-balancer-controller \ --role-name AmazonEKSLoadBalancerControllerRole \ --attach-policy-arn=arn:aws:iam::111122223333:policy/AWSLoadBalancerControllerIAMPolicy \ --approve

# Install AWS Load Balancer Controller

run > helm repo add eks https://aws.github.io/eks-charts

update the repo helm repo update eks

# Install the AWS Load Balancer Controller.

First create IAM OIDC for your cluster, run > eksctl utils associate-iam-oidc-provider --cluster my-cluster --approve

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \ -n kube-system \ --set clusterName=my-cluster \ --set serviceAccount.create=false \ --set serviceAccount.name=aws-load-balancer-controller Verify that the controller is installed

kubectl get deployment -n kube-system aws-load-balancer-controller

you'll see something like:

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE aws-load-balancer-controller   2/2     2            2           84s

Succesfully created servic eaccount and load balancer.

# LiveKit
To install livekit attach a values.yaml file.

Fore more details https://docs.livekit.io/home/self-hosting/kubernetes/

helm install <INSTANCE_NAME> livekit/livekit-server --namespace <NAMESPACE> --values values.yaml

In order to set up TURN/TLS and HTTPS on the load balancer, you may need to import your SSL certificate(s) into as a Kubernetes Secret. This can be done with:

kubectl create secret tls <NAME> --cert <CERT-FILE> --key <KEY-FILE> --namespace <NAMESPACE>

Note, please ensure that the secret is created in the same namespace as the deployment.

If you have the certificate of SSL on ACM then you don't have to do this as it will auto discover the SSL and add with it.

helm repo update helm upgrade <INSTANCE_NAME> livekit/livekit-server --namespace <NAMESPACE> --values values.yaml

If any configuration has changed, you may need to trigger a restart of the deployment. Kubernetes triggers a restart only when the pod itself has changed, but does not when the changes took place in the ConfigMap.

# Firewall & Security Groups
AWS EKS dashboard -> go to cluster details, click security group -> setup inbound rules:  https://docs.livekit.io/home/self-hosting/ports-firewall/#firewall
