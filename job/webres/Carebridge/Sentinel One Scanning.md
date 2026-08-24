# Threat Detection for Datastores - S3

## Overview

Threat Detection for Datastores - S3 protects your cloud environment with the SentinelOne Static AI and Cloud Threat Intelligence engines that inspect all of the objects in your S3 buckets. Threat Detection for Datastores - S3 automatically scans every new object added to your bucket for malware and lets you scan all objects on demand. When a malicious object is detected, Threat Detection for Datastores - S3 automatically sends it to a quarantined bucket. Threats detected by Threat Detection for Datastores - S3 show in the Threats page of the Management Console.

You can manually install the Threat Detection for Datastores - S3 Agent on your buckets, or you can use an Amazon CloudFormation template. After installing the Agent, you can set a cloud policy in the SentinelOne Management Console to customize your protection.

If you have buckets that are encrypted with Amazon's server-side encryption, you can configure AWS Key Management Service (KMS) keys to let Threat Detection for Datastores - S3 scan your encrypted files.

## How It Works

The Threat Detection for Datastores - S3 Agent is deployed to an Amazon Elastic Kubernetes Service (EKS) cluster or Amazon Elastic Container Service (ECS) cluster in your cloud environment. The Threat Detection for Amazon S3 agent needs to be deployed on a scalable, managed container cluster. This makes it easier to manage and scale the agent as it scans objects in your S3 buckets. ECS deployment, the default in newer versions of the agent, is simpler and faster than EKS deployment.

When you deploy through ECS, you don't need to manage the underlying infrastructure as you would with EKS, which requires managing a dedicated Kubernetes cluster. In addition, the ECS deployment is designed to be rapid, allowing you to deploy to a single Region in minutes.

S3 bucket event notifications are configured on buckets that match the criteria specified in the cloud policy. Event notifications relating to suspicious or malicious objects are sent to an Amazon Simple Queue Service (SQS) queue. EKS and ECS integrate with SQS to monitor event notifications from S3 buckets. This event-driven scanning is essential for real-time threat detection as new objects are added to the buckets.

The Threat Detection for Datastores - S3 Agent monitors the SQS queue events and scans new files as they are added.

If an object is suspicious or malicious, the Agent encrypts the object and moves it to a quarantined bucket where it is safe and inaccessible by other users.

Threats detected by Threat Detection for Datastores - S3 show in the Threats page of the Management Console.

## Technical Details

### Support requirements

- **Licenses:** Threat Detection for Datastores
- **Management Console:** S-24.3.1+
- **Singularity™ Operations Center:** S-24.3.1+
- **Scanner:** Threat Detection for Datastores 24.3+
- **Environment:** Standard SaaS

Learn more about support requirements.

This article explains how to deploy the Threat Detection for Datastores scanner to your Amazon Simple Storage Service (S3) buckets through Amazon Elastic Container Service (ECS).

Before starting these steps, we recommend reviewing System requirements for Threat Detection for Datastores.

### Prerequisites

Before you can deploy the scanner, you need an Amazon Virtual Private Cloud (VPC) ID and subnet ID. You can copy these values from the Amazon VPC console.

- **VPC ID** - The ID of the VPC to which you will deploy the scanner.
- **Subnet ID** - The ID of the subnet to which you will deploy the scanner. Requirements differ based on subnet type:
  - **Public subnet (internet gateway)** - The `AssignPublicIp` parameter must be set to `True`.
  - **Private subnet with access to internet (NAT gateway)** - No additional requirements.
  - **Private subnet without access to internet (No NAT gateway)** - The subnet must have these VPC endpoints attached to enable basic functionality:
    - `ecs`
    - `elasticfilesystem`
    - `s3`
    - `secretsmanager`
    - `sqs`
    - `sts`

    These VPC endpoints must be attached to enable advanced troubleshooting:
    - `logs`
    - `ssm`

The VPC and subnet should allow outgoing and incoming traffic for Threat Detection for Datastores - S3 based on Required services and ports for Console communication.

You can deploy Threat Detection for Amazon S3 with ECS by using AWS CloudFormation or Terraform.

### Deploying with CloudFormation:

1. At the top left of the Console, click the arrow to open the Scopes panel and select a scope.
2. Select a Site or an Account.
3. Click **Settings > Integrations > AWS Accounts**.
4. Ensure that the AWS cloud account, or organization you want to protect, is onboarded and healthy. To onboard your cloud account, see Connecting AWS accounts for CWS in the Management Console.
5. Click **Deploy Threat Detection for S3**.
6. Select the type of Account to which to deploy Threat Detection for Amazon S3:
   - Account for Organization
   - Standalone Account
7. In **Account**, select the account to which to deploy Threat Detection for Datastores - S3.
8. Click **Next**.
	The Deploy Threat Detection for Datastores - S3 window opens.
9. For **1. Select a template**, select **AWS CloudFormation**.
10. If you are not already logged in to your AWS account, log in and select the Region to which to deploy Threat Detection for Amazon S3.
11. Click **Launch Stack**.
    The AWS console opens to the Quick create stack wizard.
12. In the AWS Quick create stack window, enter the required **Network** parameters for the deployment.
    - **VPCId** - Enter the ID of the VPC to which to deploy Threat Detection for Datastores - S3.
    - **SubnetId** - Enter the ID of the subnet to which to deploy Threat Detection for Datastores - S3.
    - **AssignPublicIp** - Select `false` if deploying to a private subnet. Select `true` if deploying to a public subnet.

13. You can also configure some optional parameters in **CDS - Autoscaling**:


| Field                             | Description                                                                        | Default setting |
| --------------------------------- | ---------------------------------------------------------------------------------- | --------------- |
| `AutoscalingEnabled`              | Enables or disables autoscaling.                                                   | `true`          |
| `ScannerAutoscalingMinCapacity`   | Sets the minimum number of workers to deploy for on-access monitoring.             | `1`             |
| `ScannerAutoscalingMaxCapacity`   | Sets the maximum number of workers to deploy for on-access monitoring.             | `10`            |
| `FSScannerAutoscalingMinCapacity` | Sets the minimum capacity of workers to deploy for on-demand full-bucket scanning. | `0`             |
| `FSScannerAutoscalingMaxCapacity` | Sets the maximum capacity of workers to deploy for on-demand full-bucket scanning. | `10`            |

14. If you are deploying to an environment that requires a proxy, configure the **CDS - Proxy** settings:

| Field          | Description                                                                                                                     |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| `HttpProxy`    | Configures an HTTP proxy for workload containers. Use the format `http://USER:PASS@HOST:PORT`, where `USER` and `PASS` are optional. |
| `HttpsProxy`   | Configures an HTTPS proxy for workload containers. Use the format `https://USER:PASS@HOST:PORT`, where `USER` and `PASS` are optional. |
| `NoProxy`      | Configures the no-proxy parameter for workload containers. Use the format `host1,host2,host3`.                                |

15. Click **I acknowledge that AWS CloudFormation might create IAM resources with custom names**, and then click **Create stack**.

    The deployment can take several minutes to complete. You can see the status of your deployment under **Stacks** on the CloudFormation console.
### Deploying with Terraform:

1. At the top left of the Console, click the arrow to open the Scopes panel and select a scope.
2. Select a Site or an Account.
3. Click **Settings > Integrations > AWS Accounts**.
   The Deploy Threat Detection for Amazon S3 window opens.
4. Select the type of account to which to deploy Threat Detection for Datastores - S3:
   - Account for Organization
   - Standalone Account

   For more information, see Connecting AWS accounts for CWS in the Management Console.

5. In **Account**, select the account to which to deploy Threat Detection for Datastores - S3.
6. Click **Next**.

   The Deploy Threat Detection for Datastores - S3 window opens.

7. In **1. Select a template**, select **Terraform**.
8. Click **Download**.

   The archived folder downloads to your local downloads folder.

9. In your downloads folder, navigate to the downloaded archived folder, and extract the files.
10. Open the extracted folder in your terminal or command line. For example, if the folder is named `cds 111`, run the following command:

    ```bash
    cd ~/Downloads/cds\ 111/
    ```

11. Run a command to view the file contents.

    ```bash
    ls
    ```

12. Run `terraform init` to initialize the providers that terraform needs to deploy the resources needed for the Threat Detection for Datastores - S3 Agent.

    ```bash
    terraform init
    ```

    Terraform initialization starts and usually lasts several minutes. When the initialization is complete, you will see the message `Terraform has been successfully initialized!` After Terraform has been successfully initialized, you can export your AWS credentials for the account.
13. Optional: Configure the variables in the Terraform deployment to override the default settings and optimize the deployment for your needs.
14. When Terraform has been successfully initialized, enter your AWS credentials or profile in the terminal.
15. Run the `terraform apply` command to proceed with the deployment.

    ```bash
    terraform apply
    ```

    Terraform prompts for required parameters.

16. Use the values you copied from the Amazon VPC dashboard for the required parameters.
    - `var.subnet_id` - Enter the subnet ID to which to deploy Threat Detection for Datastores - S3.
    - `var.vpc_id` - Enter the ID of the VPC to which to deploy Threat Detection for Datastores - S3.
    - `var.assign_public_ip` - Set to `false` if deploying into a private subnet. Set to `true` if deploying into a public subnet.

17. After you enter the parameters, Terraform shows you an overview of the resources it will create as part of deployment.
18. For the prompt, `Do you want to perform these actions?`, enter `yes`.
    Terraform runs, creates the required resources, and deploys the Agent to Amazon S3 buckets. This process can take several minutes.


## Sources
- [Threat Detection for Datastores - S3 overview](https://community.sentinelone.com/s/article/000006085)
- [Deploying Threat Detection for Datastores scanner to S3 with ECS](https://community.sentinelone.com/s/article/000010758)