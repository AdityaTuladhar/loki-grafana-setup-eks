EKS Cluster Deployment Guide
This guide provides step-by-step instructions to deploy the provided files to an Amazon EKS cluster using eksctl and kubectl.

Prerequisites
AWS CLI configured with appropriate credentials.
eksctl installed (brew install eksctl or equivalent).
kubectl installed (brew install kubectl or equivalent).



AWS IAM permissions to create and manage EKS clusters, node groups, and IAM roles.

Deployment Steps

1. Create the EKS Cluster

    Run the following command to create an EKS cluster with a managed node group:
     
    `eksctl create cluster --name my-eks-cluster --region us-east-1 --nodegroup-name my-nodegroup --node-type t2.small --nodes 3 --nodes-min 1 --nodes-max 5 --managed`
    
    This command:
    Creates a cluster named my-eks-cluster in the us-east-1 region.
    Sets up a managed node group (my-nodegroup) with 3 t2.small nodes (scalable between 1 and 5 nodes).
    Wait for the cluster creation to complete (this may take 10-15 minutes).

2. Verify kubectl Context

    Check the current kubectl context to ensure it points to the newly created cluster:
    
    `kubectl config get-contexts`
    
    The active context should include the ARN of the cluster (e.g., arn:aws:eks:us-east-1:...:cluster/my-eks-cluster).
    
    If the context is incorrect:
    Verify that the KUBECONFIG environment variable points to the correct file. For example, if using a different Kubernetes setup like k3s, it might be:
    
    `export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`
    
    If the correct context is missing, add the cluster's kubeconfig:
    
    `eksctl utils write-kubeconfig --cluster=my-eks-cluster --region=us-east-1`

3. Update IAM Role for EBS Volumes

    To allow EC2 instances in the node group to attach EBS volumes, update the IAM role associated with the EC2 instances:

    Identify the IAM role used by the node group (check the my-nodegroup configuration in the AWS Management Console or via aws eks describe-nodegroup).

    Attach the following AWS managed policy to the role:
    
    `AmazonEBSCSIDriverPolicy`
    
    Alternatively, create a custom policy with permissions for ec2:AttachVolume, ec2:DetachVolume, and related EBS actions, then attach it to the role.

    ![image](https://github.com/user-attachments/assets/73354cd8-3197-472b-af16-0bec590aebb1)

    
    You can update the role using the AWS CLI or AWS Management Console.

4. Update AWS Tokens in Secrets File

    Edit the secrets.yaml file (or equivalent) in the repository to include the correct AWS tokens or credentials:
    
    Replace placeholder values (e.g., AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY) with valid credentials.
    
    Ensure the secrets are base64-encoded if required by the Kubernetes secrets format.
    
    Example:
    
     ```apiVersion: v1
      kind: Secret
      metadata:
        name: aws-credentials
      type: Opaque
      data:
        aws_access_key_id: <base64-encoded-key>
        aws_secret_access_key: <base64-encoded-secret>
     ```

5. Apply Deployments

    Deploy the Kubernetes resources to the cluster by applying all configuration files in the current directory:
    
    `kubectl apply -f .`

    This command applies all .yaml files (e.g., deployments, services, secrets) to the my-eks-cluster cluster.


Verification

Check that the pods are running:

`kubectl get pods`

Verify services are accessible (if applicable):

`kubectl get services`

Troubleshoot any issues by checking logs:

`kubectl logs <pod-name>`


Cleanup

To delete the cluster and avoid incurring costs:

`eksctl delete cluster --name my-eks-cluster --region us-east-1`



Notes

Ensure your AWS credentials have sufficient permissions for all operations.

The t2.small instance type is used for cost efficiency but may need adjustment for production workloads.
