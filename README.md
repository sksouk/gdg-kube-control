# create cluster using eksctl
eksctl create cluster -f cluster-build-stage.yaml

# connect to kube cluster
aws eks --region ap-southeast-1 update-kubeconfig --name [cluster_name]

------------------------------------------------------------------------------------------------
------------------------------------------------------------------------------------------------

# *important: setup ALB follow this guide (Master node should install aws-load-balancer-controller to create ALB)
https://blog.sivamuthukumar.com/aws-load-balancer-controller-on-eks-cluster 

------------------------------------------------------------------------------------------------
## Add the EKS chart repo to helm
helm repo add eks https://aws.github.io/eks-charts

## Install the AWS Load Balancer Controller CRDs - Ingress Class Params and Target Group Bindings
kubectl apply -k "github.com/aws/eks-charts/stable/aws-load-balancer-controller//crds?ref=master"

## Install the helm chart by passing the serviceAccount.create=false adn serviceAccount.name=aws-load-balancer-controller
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system --set clusterName=[cluster_name] --set serviceAccount.create=false --set serviceAccount.name=aws-load-balancer

------------------------------------------------------------------------------------------------

# deploy service
kubectl apply -f [deployment_name].yml

# delete pod
kubectl delete pod [service_name] -n [namespace]

# Get worker node
eksctl get labels --cluster [cluster_name] --nodegroup [nodegroup_name]

# Update worker node
eksctl scale nodegroup --name=[nodegroup_name] --cluster=[cluster_name] --nodes=2 --nodes-min=2 --nodes-max=10

# Get cluster
eksctl get clusters

# Get nodegroup from cluster
eksctl get nodegroup --cluster=[cluster_name]

# Delete nodegroup
eksctl delete nodegroup --cluster=[cluster_name] --name=[nodegroup_name]

# Delete cluster
eksctl delete cluster [cluster_name]

----------------------------------------------------------------
----------------------------------------------------------------
# Build image and push to ECR
docker build -t [service_name] . --platform=linux/amd64

docker tag [service_name]:latest [user_role_id].dkr.ecr.ap-southeast-1.amazonaws.com/[service_name]:latest

docker push [user_role_id].dkr.ecr.ap-southeast-1.amazonaws.com/[service_name]:latest
