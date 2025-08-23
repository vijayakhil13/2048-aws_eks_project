# 2048 Game

A web-based implementation of the popular 2048 puzzle game, containerized and deployed on AWS EKS with Fargate and exposed via AWS Application Load Balancer.

## Table of Contents

- [Demo](#demo)
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Execution Steps](#execution-steps)
- [Contributing](#contributing)
- [License](#license)
- [Contact](#contact)

## Demo

Screenshot or link to live demo (if available).

## Features

- Classic 2048 gameplay
- Simple, intuitive controls (arrow keys or swipe)
- Responsive design for desktop and mobile
- Score tracking
- Restart and undo options

## Installation

Clone the repository:

git clone https://github.com/vijayakhil13/2048-game.git
cd 2048-game





Or open `index.html` in your browser if it’s a static web project.

## Execution Steps

1. Configure AWS CLI with IAM User

aws configure


2. Create EKS Cluster (with Fargate)


eksctl create cluster --name demo-cluster --region us-east-1 --fargate
<img width="982" height="981" alt="Screenshot 2025-08-22 110726" src="https://github.com/user-attachments/assets/6aed1659-b15a-4a68-acfc-831bf4e100b9" />

3. Configure OIDC Provider


export cluster_name=demo-cluster
oidc_id=$(aws eks describe-cluster --name $cluster_name --query "cluster.identity.oidc.issuer" --output text | cut -d '/' -f 5)
<img width="969" height="616" alt="Screenshot 2025-08-22 220541" src="https://github.com/user-attachments/assets/be1842b9-6df5-4a92-a0fa-de6dca73404f" />


4. Create Fargate Profile for Namespace


eksctl create fargateprofile \
    --cluster demo-cluster \
    --region us-east-1 \
    --name alb-sample-app \
    --namespace game-2048


5. Deploy Application


kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/examples/2048/2048_full.yaml
<img width="906" height="987" alt="Screenshot 2025-08-22 110648" src="https://github.com/user-attachments/assets/2148153d-2932-4c30-a897-7062067e342c" />



6. Create IAM Policy and Integrate with Service Account


curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.11.0/docs/install/iam_policy.json

aws iam create-policy \
    --policy-name AWSLoadBalancerControllerIAMPolicy \
    --policy-document file://iam_policy.json

eksctl create iamserviceaccount \
  --cluster=<your-cluster-name> \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<your-aws-account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
<img width="1150" height="974" alt="Screenshot 2025-08-22 110708" src="https://github.com/user-attachments/assets/1e00a7af-2996-4e46-bdcb-d9665d3c55c4" />

7. Deploy ALB Ingress Controller


helm repo add eks https://aws.github.io/eks-charts
helm repo update eks
helm install aws-load-balancer-controller eks/aws-load-balancer-controller -n kube-system \
  --set clusterName=<your-cluster-name> \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=<your-region> \
  --set vpcId=<your-vpc-id>

kubectl get deployment -n kube-system aws-load-balancer-controller
<img width="1087" height="820" alt="Screenshot 2025-08-23 210650" src="https://github.com/user-attachments/assets/89b71b0c-492c-46ba-869a-d6bc9ad2c9b3" />



8. Delete EKS Cluster (Cleanup)


eksctl delete cluster --name demo-cluster --region us-east-1


## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact

Maintainer: [vijayakhil13](https://github.com/vijayakhil13)

---

Enjoy playing 2048 and feel free to suggest improvements!
