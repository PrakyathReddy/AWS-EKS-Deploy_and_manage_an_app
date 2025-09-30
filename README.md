# AWS-EKS: Deploy and manage the Game 2048

1. install kubectl - https://kubernetes.io/docs/tasks/tools/install-kubectl-macos/#install-with-homebrew-on-macos:~:text=rm%20kubectl.sha256-,Install%20with%20Homebrew%20on%20macOS,-If%20you%20are

2. install eksctl - https://docs.aws.amazon.com/eks/latest/eksctl/installation.html#:~:text=from%20Git%20Bash.-,Homebrew,-You%20can%20use

3. install AWS CLI - https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html#:~:text=Linux-,macOS,-Install%20and%20update

4. Create EKS cluster

% eksctl create cluster --name app-cluster --region us-east-1 --fargate
2025-09-30 20:40:13 [ℹ] eksctl version 0.214.0
2025-09-30 20:40:13 [ℹ] using region us-east-1
2025-09-30 20:40:15 [ℹ] setting availability zones to [us-east-1f us-east-1b]
2025-09-30 20:40:15 [ℹ] subnets for us-east-1f - public:192.168.0.0/19 private:192.168.64.0/19
2025-09-30 20:40:15 [ℹ] subnets for us-east-1b - public:192.168.32.0/19 private:192.168.96.0/19
2025-09-30 20:40:15 [ℹ] using Kubernetes version 1.32
2025-09-30 20:40:15 [ℹ] creating EKS cluster "app-cluster" in "us-east-1" region with Fargate profile
2025-09-30 20:40:15 [ℹ] if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --cluster=app-cluster'
2025-09-30 20:40:15 [ℹ] Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "app-cluster" in "us-east-1"
2025-09-30 20:40:15 [ℹ] CloudWatch logging will not be enabled for cluster "app-cluster" in "us-east-1"
2025-09-30 20:40:15 [ℹ] you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-1 --cluster=app-cluster'
2025-09-30 20:40:15 [ℹ] default addons vpc-cni, kube-proxy, coredns, metrics-server were not specified, will install them as EKS addons
2025-09-30 20:40:15 [ℹ]  
2 sequential tasks: { create cluster control plane "app-cluster",
3 sequential sub-tasks: {
1 task: { create addons },
wait for control plane to become ready,
create fargate profiles,
}
}
2025-09-30 20:40:15 [ℹ] building cluster stack "eksctl-app-cluster-cluster"
2025-09-30 20:40:17 [ℹ] deploying stack "eksctl-app-cluster-cluster"
2025-09-30 20:40:47 [ℹ] waiting for CloudFormation stack "eksctl-app-cluster-cluster"
....
2025-09-30 20:50:40 [ℹ] creating addon: vpc-cni
2025-09-30 20:50:41 [ℹ] successfully created addon: vpc-cni
2025-09-30 20:50:42 [ℹ] creating addon: kube-proxy
2025-09-30 20:50:43 [ℹ] successfully created addon: kube-proxy
2025-09-30 20:50:43 [ℹ] creating addon: coredns
2025-09-30 20:50:44 [ℹ] successfully created addon: coredns
2025-09-30 20:50:45 [ℹ] creating addon: metrics-server
2025-09-30 20:50:46 [ℹ] successfully created addon: metrics-server
2025-09-30 20:52:50 [ℹ] creating Fargate profile "fp-default" on EKS cluster "app-cluster"
2025-09-30 20:53:25 [ℹ] created Fargate profile "fp-default" on EKS cluster "app-cluster"
2025-09-30 20:53:57 [ℹ] "coredns" is now schedulable onto Fargate
2025-09-30 20:55:04 [ℹ] "coredns" is now scheduled onto Fargate
2025-09-30 20:55:04 [ℹ] "coredns" pods are now scheduled onto Fargate
2025-09-30 20:55:04 [ℹ] waiting for the control plane to become ready
2025-09-30 20:55:07 [✔] saved kubeconfig as "/Users/praky/.kube/config"
2025-09-30 20:55:07 [ℹ] no tasks
2025-09-30 20:55:07 [✔] all EKS cluster resources for "app-cluster" have been created
2025-09-30 20:55:24 [ℹ] kubectl command should work with "/Users/praky/.kube/config", try 'kubectl get nodes'
2025-09-30 20:55:24 [✔] EKS cluster "app-cluster" in "us-east-1" region is ready

5. to enable direct use of kubectl
   % aws eks update-kubeconfig --name app-cluster --region us-east-1
   Added new context arn:aws:eks:us-east-1:331221168851:cluster/app-cluster to /Users/praky/.kube/config

6. create fargate profile - this is because we want to use a new namespace to deploy this app
   % eksctl create fargateprofile \

   > --cluster app-cluster \
   > --region us-east-1 \
   > --name 2048-app \
   > --namespace game-2048
   > 2025-09-30 22:06:59 [ℹ] creating Fargate profile "2048-app" on EKS cluster "app-cluster"
   > 2025-09-30 22:07:18 [ℹ] created Fargate profile "2048-app" on EKS cluster "app-cluster"

7. Deploy the 2048 game using a pre-existing template -
   % kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
   namespace/game-2048 created
   deployment.apps/deployment-2048 created
   service/service-2048 created
   ingress.networking.k8s.io/ingress-2048 created

It creates namespace, deployment with the 2048 image on 5 replicas, creates a service and then an ingress. Ingress will not work at this point simply because there's no ingress controller yet.

% k get ingress -A
NAMESPACE NAME CLASS HOSTS ADDRESS PORTS AGE
game-2048 ingress-2048 alb \* 80 5m34s

8. create ingress controller which will then read the ingress resource called ingress-2048 and then create a load balancer for us along with configuration of target group, ports to access, etc.

Deploy ALB ingress controller

As a pre-requisite, first setup OIDC connector
Configure IAM OIDC provider

$ export cluster_name=demo-cluster
$ oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
$ eksctl utils associate-iam-oidc-provider --cluster $cluster_name --approve
2025-09-30 22:33:35 [ℹ] will create IAM Open ID Connect provider for cluster "app-cluster" in "us-east-1"
2025-09-30 22:33:36 [✔] created IAM Open ID Connect provider for cluster "app-cluster" in "us-east-1"

9. Install ALB controller which is just a pod. This pod should be granted access to AWS services such as ALB
   First create an IAM policy and role for this
   Download IAM policy: curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

Create IAM policy
aws iam create-policy \
 --policy-name AWSLoadBalancerControllerIAMPolicy \
 --policy-document file://iam_policy.json
{
"Policy": {
"PolicyName": "AWSLoadBalancerControllerIAMPolicy",
"PolicyId": "ANPAU2HSR6LJSSUQFNZAO",
"Arn": "arn:aws:iam::331221168851:policy/AWSLoadBalancerControllerIAMPolicy",
"Path": "/",
"DefaultVersionId": "v1",
"AttachmentCount": 0,
"PermissionsBoundaryUsageCount": 0,
"IsAttachable": true,
"CreateDate": "2025-09-30T17:14:09+00:00",
"UpdateDate": "2025-09-30T17:14:09+00:00"
}
}

Create IAM role
eksctl create iamserviceaccount \
 --cluster=app-cluster \
 --namespace=kube-system \
 --name=aws-load-balancer-controller \
 --role-name AmazonEKSLoadBalancerControllerRole \
 --attach-policy-arn=arn:aws:iam::331221168851:policy/AWSLoadBalancerControllerIAMPolicy \
 --approve
service account is getting created and attached to the role

We will then use the same service account (iamserviceaccount) in the application

10. create application load balancer - ALB controller. Will use helm chart that will create the actual controller using service account created earlier

install helm: $ brew install helm

$ helm repo add eks https://aws.github.io/eks-charts
$ helm repo update eks

Install the helm chart

$ helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
 --set clusterName=app-cluster \
 --set serviceAccount.create=false \
 --set serviceAccount.name=aws-load-balancer-controller \
 --set region=us-east-1 \
 --set vpcId=vpc-0637b5825d9256cee

vpc id from cluster > networking > VPC

NAME: aws-load-balancer-controller
LAST DEPLOYED: Tue Sep 30 22:57:25 2025
NAMESPACE: kube-system
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
AWS Load Balancer controller installed!

$ k get deploy -n kube-system  
NAME READY UP-TO-DATE AVAILABLE AGE
aws-load-balancer-controller 2/2 2 2 106s

The load balancer controller deployment creates 2 replicas, one in each availability zone, continuously watches for ingress resources and creates ALB resources in 2 AZ's

Now with the ALB load balancer successfully up and running, ingress resource will get a valid address

$ k get ingress -A  
NAMESPACE NAME CLASS HOSTS ADDRESS PORTS AGE
game-2048 ingress-2048 alb \* k8s-game2048-ingress2-c7d1378cbc-912773785.us-east-1.elb.amazonaws.com 80 54m

This address is that of the load balancer that the ingress controller has created after watching and acting on the ingress resource

success !!

App now available for public access on:
http://k8s-game2048-ingress2-c7d1378cbc-912773785.us-east-1.elb.amazonaws.com/

Source: https://youtu.be/RRCrY12VY_s?si=RsB--pmpzzbH27Jc
Ref github: https://github.com/iam-veeramalla/aws-devops-zero-to-hero/tree/main/day-22
