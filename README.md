# EC2 Deployment — CloudFormation Template

A sanitised, reusable CloudFormation template for deploying a production-grade EC2 instance with encrypted EBS volumes, an Elastic IP, and a CloudWatch alarm for system status monitoring.

No hardcoded secrets, credentials, or account-specific values. All environment-specific configuration is passed in as parameters at deploy time.

---

## What this template does

Deploys the following resources into your AWS account:

- **EC2 instance** — configurable instance type (defaults to `t3a.medium`), with termination protection enabled
- **Two encrypted EBS volumes** — a 100 GB root volume (`/dev/sda1`) and a 40 GB data volume (`/dev/xvdf`), both `gp3`, both with `DeleteOnTermination: false`
- **Elastic IP** — a static public IP associated with the instance
- **CloudWatch alarm** — monitors `StatusCheckFailed_System` and notifies via SNS if the system status check fails for two consecutive periods

All volumes are encrypted at rest. The instance is tagged with `Name`, `Project`, `Backup`, and `Environment` for easy filtering and cost allocation.

---

## Deploying via the AWS Console

### Prerequisites

Before deploying, make sure you have the following available in your target AWS region:

| Requirement | Notes |
|---|---|
| An AMI ID | Must exist in the target region |
| A VPC subnet ID | The instance will be launched into this subnet |
| One or more security group IDs | Must belong to the same VPC as the subnet |
| An EC2 key pair name | For SSH access |
| An IAM instance profile name | The profile must exist and have appropriate permissions |
| An SNS topic ARN | Used for CloudWatch alarm notifications |

### Steps

1. Log in to the [AWS Console](https://console.aws.amazon.com) and navigate to **CloudFormation**
2. Click **Create stack** → **With new resources (standard)**
3. Under *Specify template*, choose **Upload a template file** and select `Git_IAC.json`
4. Click **Next** and fill in the parameters:

| Parameter | Description |
|---|---|
| `AmiId` | AMI ID to use (e.g. `ami-0abcdef1234567890`) |
| `SubnetId` | Subnet to launch the instance into |
| `SecurityGroupIds` | Comma-separated list of security group IDs |
| `KeyPairName` | Name of your EC2 key pair |
| `IamInstanceProfile` | Name of the IAM instance profile to attach |
| `SnsAlarmTopicArn` | Full ARN of the SNS topic for alerts |
| `InstanceType` | Instance type — defaults to `t3a.medium` |
| `ServerName` | Short hostname prefix, max 8 alphanumeric characters |
| `Project` | Project or customer identifier used for tagging |

5. Click **Next**, optionally add stack tags, then click **Next** again
6. Review the configuration and click **Submit**

The stack typically completes in 2–3 minutes. Once complete, the **Outputs** tab will show the instance ID and the assigned Elastic IP.

---

## Outputs

| Output | Description |
|---|---|
| `InstanceId` | The EC2 instance ID |
| `PublicIP` | The Elastic IP address assigned to the instance |

---

## Notes

- Termination protection is enabled by default. To delete the stack you must first disable it manually under **EC2 → Instances → Change termination protection**
- Both EBS volumes have `DeleteOnTermination: false` — they persist after stack deletion and must be cleaned up manually if no longer needed
- The CloudWatch alarm fires after two consecutive 60-second periods of system status check failure

---

## License

MIT
