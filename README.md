# EKS Terraform Configuration for Halloween Game

This repository contains Terraform configuration to create an AWS EKS (Elastic Kubernetes Service) cluster for deploying the Halloween Candy Rush game.

## Architecture

The configuration creates:
- VPC with public and private subnets across 3 availability zones
- EKS cluster version 1.33 named `hl-game-cluster`
- EKS managed node group with auto-scaling (1-3 nodes, t3.medium instances)
- nginx-ingress controller with Classic Load Balancer

## Prerequisites

1. **AWS CLI** installed and configured with profile `claude-aws`
   ```bash
   aws configure --profile claude-aws
   ```

2. **Terraform** (version >= 1.0)
   ```bash
   # Install Terraform
   wget https://releases.hashicorp.com/terraform/1.6.0/terraform_1.6.0_linux_amd64.zip
   unzip terraform_1.6.0_linux_amd64.zip
   sudo mv terraform /usr/local/bin/
   ```

3. **kubectl** for Kubernetes management
   ```bash
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```

4. **Sufficient AWS permissions** for creating:
   - VPC, subnets, NAT Gateway, Internet Gateway
   - EKS clusters and node groups
   - EC2 instances, security groups
   - IAM roles and policies
   - Elastic Load Balancers

## Deployment Instructions

### 1. Initialize Terraform

```bash
terraform init
```

This will download all required providers and modules.

### 2. Review the Plan

```bash
terraform plan
```

Review the resources that will be created. You can customize variables in `variables.tf` or create a `terraform.tfvars` file.

### 3. Apply Configuration

```bash
terraform apply
```

Type `yes` when prompted. The deployment will take approximately 15-20 minutes.

### 4. Configure kubectl

After successful deployment, configure kubectl to interact with your cluster:

```bash
aws eks --region us-east-1 update-kubeconfig --name hl-game-cluster --profile claude-aws
```

Or use the output command:
```bash
terraform output -raw configure_kubectl | bash
```

### 5. Verify Installation

Check cluster nodes:
```bash
kubectl get nodes
```

Check nginx-ingress controller:
```bash
kubectl get svc -n ingress-nginx
```

Get the Load Balancer hostname:
```bash
kubectl get svc -n ingress-nginx nginx-ingress-ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```

## Configuration

### Default Variables

- **AWS Region**: `us-east-1`
- **AWS Profile**: `claude-aws`
- **Cluster Name**: `hl-game-cluster`
- **Kubernetes Version**: `1.33`
- **Node Instance Type**: `t3.medium`
- **Node Group Size**: 1-3 nodes (desired: 2)

### Customization

Create a `terraform.tfvars` file to override defaults:

```hcl
aws_region              = "eu-west-1"
node_instance_types     = ["t3.large"]
node_group_desired_size = 3
```

## Outputs

After deployment, Terraform provides:
- **cluster_endpoint**: EKS API server endpoint
- **cluster_name**: Name of the EKS cluster
- **configure_kubectl**: Command to configure kubectl
- **nginx_ingress_service**: Command to get load balancer hostname

View outputs:
```bash
terraform output
```

## nginx-ingress Controller

The configuration automatically installs nginx-ingress controller with:
- Classic Load Balancer (not NLB/ALB)
- Cross-zone load balancing enabled
- Deployed in `ingress-nginx` namespace

## Cleanup

To destroy all created resources:

```bash
terraform destroy
```

**Warning**: This will delete the EKS cluster and all associated resources. Make sure to backup any important data.

## Cost Estimation

Approximate monthly costs (us-east-1):
- EKS cluster: ~$73/month
- 2x t3.medium nodes: ~$60/month
- NAT Gateway: ~$32/month
- Classic Load Balancer: ~$18/month
- **Total**: ~$183/month

## Troubleshooting

### Issue: Terraform fails with authentication error
**Solution**: Ensure AWS profile `claude-aws` is configured:
```bash
aws sts get-caller-identity --profile claude-aws
```

### Issue: EKS cluster creation timeout
**Solution**: EKS cluster creation can take 15-20 minutes. If it times out, check AWS console for cluster status.

### Issue: kubectl cannot connect to cluster
**Solution**: Update kubeconfig and verify AWS credentials:
```bash
aws eks update-kubeconfig --name hl-game-cluster --profile claude-aws --region us-east-1
kubectl get svc
```

### Issue: nginx-ingress pods not starting
**Solution**: Check pod status and logs:
```bash
kubectl get pods -n ingress-nginx
kubectl logs -n ingress-nginx <pod-name>
```

## Next Steps

After successful deployment:
1. Configure Cloudflare DNS to point to the Load Balancer hostname
2. Deploy the Halloween game using Helm chart
3. Access the game via the configured domain

## References

- [AWS EKS Terraform Module](https://registry.terraform.io/modules/terraform-aws-modules/eks/aws/latest)
- [nginx-ingress Controller](https://kubernetes.github.io/ingress-nginx/)
- [EKS Best Practices](https://aws.github.io/aws-eks-best-practices/)