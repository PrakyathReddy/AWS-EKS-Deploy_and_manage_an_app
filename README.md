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
NAMESPACE   NAME           CLASS   HOSTS   ADDRESS   PORTS   AGE
game-2048   ingress-2048   alb     *                 80      5m34s

