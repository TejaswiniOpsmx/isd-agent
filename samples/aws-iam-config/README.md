# Instructions to Create Roles, User & Policies for AWS Integration with Spinnaker
This document will help you understand to create Managed Roles, Managing users and other aspects to Integrate AWS as a cloud provider account with Spinnaker. Following steps will be covered.

## If your Spinnaker is Installed on AWS EKS, follow the below instructions

### Steps to Create a Policy
- Follow the below process in case your Spinnaker is set up on a Kubernetes Cluster which is not managed by AWS EKS.
- Login to AWS Console, in the IAM section click on Policies and create a new Policy.
- Now, Click on Create new policy and JSON. In the JSON field add the content available [here](https://github.com/OpsMx/aws-files/blob/main/Spinnaker-Granular-Policy.json)
- Now, name the newly created policy and save it.

### Steps to Create a Managed Role
- Click on Role and Create Role and Select the use case to be EC2.
- Now, Click on Next Permissions and Select the below Policies.
    - Newly Created policy in the above step.
    - ReadOnlyAccess
    - SecretsManagerReadWrite(Optional - If applicable in your use case)
    - AmazonECSTaskExecutionRolePolicy
- Update the Trust relationships of the role from EC2 to Managing User. Sample JSON [here](https://github.com/OpsMx/aws-files/blob/main/TrustRelationship-For-EKS-Role.json)

### Steps to Create a Role and assign to EKS NodeGroup
- Now, if you have multiple AWS Accounts and have Managed Role with Similar naming convention everywhere, create a Cross-Account-Assume Policy. Refer to the file available [here](https://github.com/OpsMx/aws-files/blob/main/Cross-Account-AssumeRole-NodeGroup.json)
- Ensure to attach the below listed policies for the Node Group role to assume Permissions attached to the Managed Role.
    - cross-account-policy
    - AmazonEKSWorkerNodePolicy
    - AmazonEC2ContainerRegistryReadOnly
    - AmazonEC2ReadOnlyAccess
    - ReadOnlyAccess
    - AmazonEKS_CNI_Policy

**Note: In case, if the names are different in each AWS account then add multiple resources with their Respective AWS Account IDs in the same Cross-Account Policy.**

## If you have Installed Spinnaker on any other provider apart from EKS like, AKS, GKE, OnPrem, OpenShift. Follow the below steps to integrate AWS with Spinnaker

### Steps to Create a Policy
- Follow the below process in case your Spinnaker is set up on a Kubernetes Cluster which is not managed by AWS EKS.
- Login to AWS Console, in the IAM section click on Policies and create a new Policy.
- Now, Click on Create new policy and JSON. In the JSON field add the content available [here](https://github.com/OpsMx/aws-files/blob/main/Policy-For-Non-EKS-Role.json)
- Now, name the newly created policy and save it.

### Steps to Create a Managing User
- This Managing User creation process will be only required in case your spinnaker is installed in Kubernetes cluster which is not managed by AWS.
- After finishing the above policy creation process. Now, click on Users and click on Add User.
- Provide a Username according to your requirement and Select Programmatic Access for the User.
- Now, attach the newly created policy  in the above step by selecting it from the "Attach Existing Policies"
- Tags are Optional, provided as per requirement. Now click on Review and Create User.

### Steps to Create a Managed Role
- Click on Role and Create Role and Select the use case to be EC2.
- Now, Click on Next Permissions and Select the below Policies.
    - Newly Created policy in the above step.
    - ReadOnlyAccess
    - SecretsManagerReadWrite(Optional - If applicable in your use case)
    - AmazonECSTaskExecutionRolePolicy
- Update the Trust relationships of the role from EC2 to Managing User. Sample JSON for the TrustRelationships available [here](https://github.com/OpsMx/aws-files/blob/main/TrustRelationship-For-Non-EKS-Role.json)

## Make Subnets Visible in Spinnaker UI
- In Spinnaker, multiple subnets are grouped together into one by the above configuration tag to every subnet. The actual subnet value is not displayed on the ResourceGroup page if the purpose text is the same across the Subnets. The Spinnaker subnet value is displayed as <purpose> name.
- To make subnets visible to Spinnaker, every subnet needs to have a tag called ‘immutable_metadata’. Syntax
- Select the Subnet you want to access via Spinnaker. Click on Manage Tags and add the below content in the tag
     
        immutable_metadata={"purpose":"<example-purpose>"} 
        E.G.
        immutable_metadata={"purpose":"web-alb"}

## Steps to Configure AWS as CloudProvider with Spinnaker
- After completing the above, now execute the below commands to Integrate AWS as a Cloud Provider with Spinnaker.
  ```
  export ACCESS_KEY_ID=1234567891012
  export AWS_ACCOUNT_NAME=aws-account
  export ACCOUNT_ID=123456789012 
  export ROLE_NAME=role/ROL-Spinnaker-Managed-Role
  
  hal config provider aws account add ${AWS_ACCOUNT_NAME} \
     --account-id ${ACCOUNT_ID} \
     --assume-role ${ROLE_NAME} \ 
     --regions us-east-1
  ```
- **For Non EKS, provide access and secret keys by executing the below commands**
  ```
  hal config provider aws edit --access-key-id ${ACCESS_KEY_ID} \
     --secret-access-key # do not supply the key here, you will be prompted 
  hal config provider aws bakery edit --aws-access-key ${ACCESS_KEY_ID} \
     --aws-secret-key # do not supply the key here, you will be prompted
  ```
- Now, execute the below command to enable AWS as a cloudprovider and restart spinnaker services. By executing the below commands
  ```
  hal config provider aws enable
  hal deploy apply
  ```
  
  ***After Following these issues if you still run into any issues, feel free to comment here.***
