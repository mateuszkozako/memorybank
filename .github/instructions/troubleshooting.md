# Troubleshooting Guide

## Critical Commands & Workflows

### EKS Development
```bash
# Run specific tests
cd jeppesen-eks/test && go test -run TestAlphaEKS

# Validate Terraform
terraform plan -var-file=examples/basic.tfvars

# Check module docs
terraform-docs markdown table --output-file README.md .

# Test with different configurations
terraform plan -var-file=examples/bottlerocket.tfvars
terraform plan -var-file=examples/istio.tfvars
```

### OCM Client Testing
```bash
cd ocm-client

# Setup credentials
python tools/setup_credentials.py

# Run integration tests
python tests/integration/test_ocm_comprehensive.py

# Validate configuration
python tools/verify_config.py

# Run specific test suites
python -m pytest tests/integration/ -v
python -m pytest tests/unit/ -v
```

### FluxCD Deployment Debugging
```bash
# Check Terraform reconciliation
kubectl get terraform -A

# Check GitRepository sources
flux get sources git

# Check Terraform source status
flux get terraforms

# Suspend/resume deployments
kubectl patch terraform cluster-name --type json --patch='[{"op": "add", "path": "/spec/suspend", "value": true}]'

# Force reconciliation
flux reconcile source git jeppesen-eks
flux reconcile terraform cluster-name
```

## Common Issues & Solutions

### EKS-Specific Issues
- **EKS + EFS Driver**: Requires minimum 2 nodes for proper scheduling
- **Karpenter upgrades**: Cannot upgrade without manual CRD cleanup first
- **Version skew**: Control plane and worker node compatibility matrix
- **Node group updates**: AL2 vs Bottlerocket requires replacement, not in-place updates

### Kubernetes Version Management
- **Sequential upgrades**: Must upgrade 1.28→1.29→1.30, cannot skip versions
- **API deprecations**: Check for deprecated APIs before upgrading
- **Add-on compatibility**: Ensure EKS add-ons support target version

### GitOps and FluxCD Issues
- **Git source naming**: EKS sources must start with `jeppesen-eks` prefix
- **Resource naming**: Crossplane uses `metadata.name` as identifier
- **Terraform state**: State conflicts require manual intervention
- **Reconciliation loops**: Check for circular dependencies in manifests

### OCM Client Issues
- **Credential rotation**: Secrets Manager integration requires proper IAM permissions
- **Azure AD integration**: Ensure service principal has correct scopes
- **Function app deployment**: Cold starts may cause timeout issues
- **Environment mismatch**: Dev/UAT/Prod configurations must align

## Debug Commands

### Cluster Access and Inspection
```bash
# EKS cluster access
aws eks update-kubeconfig --name cluster-name --region us-west-2

# Check cluster status
kubectl get nodes
kubectl get pods -A

# Inspect specific resources
kubectl describe node node-name
kubectl logs -n kube-system deployment/coredns
```

### Terraform State Management
```bash
# Terraform state inspection
terraform state list
terraform state show resource.name
terraform show

# Import existing resources
terraform import aws_eks_cluster.cluster cluster-name

# Refresh state
terraform refresh
```

### FluxCD Status Checking
```bash
# Check FluxCD status
flux check

# Get all sources
flux get sources all

# Check specific components
flux get helmreleases -A
flux get kustomizations -A

# Logs and events
flux logs --level=error
kubectl get events --sort-by='.lastTimestamp'
```

### Network and Connectivity
```bash
# VPC and networking
aws ec2 describe-vpcs --filters "Name=tag:Name,Values=vpc-name"
aws ec2 describe-subnets --filters "Name=vpc-id,Values=vpc-id"

# Security groups
aws ec2 describe-security-groups --group-ids sg-xxx

# Load balancer status
kubectl get svc -A | grep LoadBalancer
aws elbv2 describe-load-balancers
```

## Performance Monitoring

### Resource Utilization
```bash
# Node resource usage
kubectl top nodes
kubectl top pods -A

# Cluster capacity
kubectl describe node | grep -A 5 "Allocated resources"

# Karpenter scaling
kubectl get nodes -l karpenter.sh/provisioner-name
kubectl logs -n karpenter deployment/karpenter
```

### Application Health
```bash
# Health checks
kubectl get pods -A --field-selector=status.phase!=Running
kubectl get events --field-selector type=Warning

# Service mesh (if Istio enabled)
istioctl proxy-status
istioctl analyze
```

## Recovery Procedures

### Cluster Recovery
1. **Backup current state**: Export critical resources
2. **Identify root cause**: Check logs, events, and metrics
3. **Incremental fixes**: Apply minimal changes first
4. **Validate each step**: Test after each change
5. **Document lessons learned**: Update runbooks

### GitOps Recovery
1. **Suspend automation**: Prevent further reconciliation
2. **Manual intervention**: Apply fixes directly if needed
3. **Update manifests**: Fix source of truth in Git
4. **Resume automation**: Re-enable FluxCD reconciliation
5. **Monitor closely**: Watch for recurring issues

### Data Recovery
1. **AWS Secrets Manager**: Use version history for credential recovery
2. **Terraform state**: Restore from backup or rebuild
3. **Persistent volumes**: Check EBS snapshots and backups
4. **Application data**: Follow app-specific recovery procedures

## Escalation Procedures

### Internal Team
- **Platform Team**: Infrastructure and GitOps issues
- **Security Team**: Credential and compliance issues
- **Application Teams**: Workload-specific problems

### External Support
- **AWS Support**: EKS, VPC, and service issues
- **HashiCorp**: Terraform-specific problems
- **CNCF Community**: Kubernetes and cloud-native issues

### Emergency Contacts
- **On-call rotation**: Check team documentation
- **Incident management**: Follow established procedures
- **Communication channels**: Slack, PagerDuty, or equivalent