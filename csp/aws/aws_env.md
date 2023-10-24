# About this document

This guide will help you deploy a microserviced application that you can then protect with a CNAPP solution for use in various lab exercises. It is designed to help you experience the different capabilities of a CNAPP and how they can all be brought to bear on a single application.

The activities in this guide can be broken down into the following phases:
- Create a Kubernetes cluster on AWS
- Deploy the application to the Kubernetes cluster

If you have already performed any of these activities previously, bring this up with your instructor, as these steps may already be redundant for you.

**Assumptions**:

This lab guide assumes that you already have the relevant Cloud Service Provider for use in this activity.

# Create a Kubernetes cluster on AWS. 

This process is broken into the following steps:

- Create an AWS IAM service role
- Create a VPC to deploy the cluster
- Create an AWS EKS cluster
- Configure kubectl for Amazon EKS
- Create worker nodes for the EKS cluster

## Create an AWS IAM service role

This creates an IAM role that allows Kubernetes to create AWS resources.

1. After logging on to the AWS console, search for Identity and Access Management (IAM)

2. Click **Roles** on the left-menu, and then at the main screen that opens, click **Create role**.

3. At the Select trusted entity step, select **AWS service**, and then **EKS - Cluster** as the use case. 

4. Click **Next** on this step and the step that follows. The default setting on the next step is sufficient for this lab.

5. Give the role a name and then click **Create role**. You will use this role in the step where you will create the EKS cluster.


## Create a VPC to deploy the cluster

1. At the AWS console, search for CloudFormation

2. Click **Create Stack** > **With new resources**

3. At the Create stack page select the following options:

4. Download this YAML file (right-click link > Save as), and then upload to the Create Stack screen and then click **Next**.

5. Give the stack a name and then click **Next**.


**Note**: Choose your stack name wisely as this will simplify identification of resources in later steps.

6. Complete the reviews in the remaining steps (the defaults will be sufficient for this lab) and then click **Create** to complete the process.

7. After completion of stack creation, go to the Outputs tab and copy the contents to a local file. This will contain the following information that you will need when creating the EKS cluster:

- SecurityGroups
- VpcId
- SubnetIds 


## Configure VPC

The steps here are not, strictly speaking, necessary for the application that you are deploying to work. However, if you are going to secure this application with a CNAPP, they are necessary to give the the necessary visibility into how the application operates.

- Create flow log
- Enable CloudTrail

### Create flow log

The following procedure assumes you will use an Amazon S3 bucket as the destination for the flow logs, and that you do not already have an S3 bucket.

**Note**: The CNAPP that will monitor this cloud account must also be configured appropriately.

1. At the VPC dashboard, select the VPC for which you will be creating flow logs and then locate the Flow logs tab on the lower part of the screen. Click **Create flow log**.

2. At the Create flow log screen, retain the defaults except for the Destination. Select **Send to an Amazon S3 bucket**.

3. Click the **Create S3 bucket** link. This will open the Amazon S3 screen on a separate tab.

4. At the screen that opens, provide a name the bucket and retain all defaults. Click **Create bucket**. This return you to the Amazon S3 bucket screen.

5. Select the bucket you just created, and then select the Properties tab. Copy the ARN name of the bucket.

6. Return to the Create flow log screen and enter the ARN of the bucket in the S3 bucket ARN field. Click **Create flow log**.

### Enable CloudTrail

To create a CloudTrail:
1. At the AWS CloudTrail console, click Create trail.
2. At the Cloud Attributes screen, choose a name for the trail. For purposes of this exercise, create a new S3 bucket.
3. Enable Log file SSE-KMS encryption. If creating a new key, provide an alias.
4. At the Choose log events screen, select Management events, and enable Read and Write API activities.
5. Click Create Trail in the final screen.

## Create an AWS EKS cluster

1. At the AWS console, search for EKS

2. Click **Create cluster** or **Add cluster** > **Create** in the main screen

3. Give the cluster a name and then select the Cluster service role created earlier and then click **Next**.

**Note**: Choose your cluster name wisely as this will simplify identification of resources in later steps

4. Provide the VpcId, SubnetIds, and SecurityGroups collected earlier and then click **Next**.

**Note**: The dropdown menus will also provide suggestions for these settings. If you selected an intuitive name in your previous steps, these will help with the selection of the appropriate information. 

5. At the "Configure logging" screen enable all logging options.

**Note**: Enabling these logs may have cost implications. This is not absolutely necessary for setting up the lab and can actually be turned on at a later point as required

6. Proceed to the remaining steps in the wizard, retaining the defaults as these will be sufficient, until you reach the final review step and then click **Create**.

## Configure kubectl for Amazon EKS

1. At the AWS console, search for CloudShell

2. Run the following command to configure kubectl for AWS EKS
```
aws eks --region <region> update-kubeconfig --name <clusterName>
```

## Create worker nodes for the EKS cluster

1. Download this YAML file (right-click link > Save as)

2. At the AWS console, return to CloudFormation and create a stack using the YAML file downloaded in the previous step.

3. At the stack details page, provide the following information:

| Parameter | Description |
| ------ | ------ |
| Stack name | Enter a unique name |
| ClusterName | Enter the name of Kubernetes cluster created earlier |
| ClusterControlPlaneSecurityGroup | Enter the SecurityGroup obtained from the Output tab earlier |
| NodeGroupName | Enter a unique name |
| NodeAutoScalingGroupMinSize |^  |
| NodeAutoScalingGroupDesiredCapacity |^ |
| NodeAutoScalingGroupMaxSize |^  |
| NodeInstanceType |^ Defaults are populated by YAML file ingested earlier. These default are sufficient for this lab |
| NodeImageId | Refer to EKS AMI document to find the correct AMI ID for your account’s region: https://docs.aws.amazon.com/eks/latest/userguide/retrieve-ami-id.html **Note**: Match the Kubernetes version in this page with the one you are using in the environment|
| KeyName | Specify an EC2 key pair to allow SSH access to instances. Follow directions here: https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-key-pairs.html |
| VpcId | Enter VpcId obtained from the Output tab in the previous step |
| Subnets | Enter SubnetIds obtained from the Output tab in the previous step |

4. Complete the reviews in the remaining steps and then click **Create** to complete the process.

5. After creating the stack view the Output tab and copy the NodeInstanceRole value. This will be used later in this lab.

6. Add worker nodes to the EKS cluster. Using CloudShell, create the following file with this CLI command shown.
```
vim ~/.kube/aws-auth-cm.yaml
```

7. Enter the following text in the YAML file above. Replace “ < ARM of instance role > ” with the NodeInstanceRole obtained from the Output tab earlier.

**Note**: The contents of the YAML file, particularly the indentation, must appear as shown below.

```
apiVersion: v1
kind: ConfigMap
metadata:
 name: aws-auth
 namespace: kube-system
data:
 mapRoles: |
   - rolearn:  <ARN of instance role>
     username: system:node:{{EC2PrivateDNSName}}
     groups:
       - system:bootstrappers
       - system:nodes
```


8. Run the following command on CloudShell:

```
kubectl apply -f ~/.kube/aws-auth-cm.yaml
```

# Deploy the application to the Kubernetes cluster

For this version of this lab, Google-supported Online Boutique will be used as the basis for the exercise. This is an open-source microserviced demo application. As of writing, integration with a CI/CD system had not yet been completed, therefore that application will be deployed manually.

**Note**: The steps in this section of the document assume that they were performed in the sequence laid out in this guide. Deployment prerequisites are explained in previous sections.

Using CloudShell, clone the components of the Online Boutique from its repository to the Kubernetes cluster

```
git clone https://github.com/GoogleCloudPlatform/microservices-demo
```

Open the microservices-demo directory and run the following command

```
kubectl apply -f ./release/kubernetes-manifests.yaml
```

Wait for the pods to be ready. Run the following command periodically

```
kubectl get pods
```

Eventually the command above will display the following

```
NAME                                     READY   STATUS    RESTARTS   AGE
adservice-76bdd69666-ckc5j               1/1     Running   0          2m58s
cartservice-66d497c6b7-dp5jr             1/1     Running   0          2m59s
checkoutservice-666c784bd6-4jd22         1/1     Running   0          3m1s
currencyservice-5d5d496984-4jmd7         1/1     Running   0          2m59s
emailservice-667457d9d6-75jcq            1/1     Running   0          3m2s
frontend-6b8d69b9fb-wjqdg                1/1     Running   0          3m1s
loadgenerator-665b5cd444-gwqdq           1/1     Running   0          3m
paymentservice-68596d6dd6-bf6bv          1/1     Running   0          3m
productcatalogservice-557d474574-888kr   1/1     Running   0          3m
recommendationservice-69c56b74d4-7z8r5   1/1     Running   0          3m1s
redis-cart-5f59546cdd-5jnqf              1/1     Running   0          2m58s
shippingservice-6ccc89f8fd-v686r         1/1     Running   0          2m58s
```

Test the application by connecting to its public IP address. Run the following command to determine this address:

```
kubectl get service frontend-external | awk '{print $4}'
```


**Note**: If the application is not immediately responsive, wait a few minutes to give the CSP time to complete network configuration.

# Cleanup 

The following instructions guide you through the process of deleting the resources that were deployed as part of this lab environment. 

**Note**: The cleanup process is essentially in the reverse order of the setup process. 

## Delete CNAPP components (if any)

If the CNAPP protecting the application involved modification of the CSP environment, undo these based on CNAPP documentation.

## Delete auto-scaling group 
Start with this step to prevent auto-scaling functionality from recreating EC2 instances that are stopped. If you neglect this step, you will receive the following error when you stop instances associated with the EKS cluster in this lab.

To delete the auto-scaling group: 

1. At the AWS console, search for EC2
 
2. At the left-menu, scroll down to Auto Scaling groups.
 
3. Select the Auto Scaling group that you created for this environment and then click **Delete**
 
4. At the following screen, confirm the intent to delete Auto Scaling Group by entering delete in the field provided.

## Delete load balancer
 
Peform this step after deleting the auto-scaling group for this application.

To delete the load balancer:

1. At the AWS console, search for VPC

2. At the left-menu, scroll down to **Load Balancing** > **Load Balancers**.

3. Select the load balancer created for the application, and then click **Actions** > **Delete load balancer**.


## Delete VPC 

This is the final step of the cleanup process. If you missed or skipped relevant steps, you will receive errors such as, but not limited to, this.

Review previous steps if you encounter issues such as the one above.  

**Note**: Undeleted resources will be reflected at your end-of-month billing statement.

To delete the VPC: 

1. At the AWS console, search for VPC

2. At the VPC screen, select the VPC that you need to delete, and then click **Actions** > **Delete VPC**
