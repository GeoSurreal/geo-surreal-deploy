# Deploy Geo Surreal with Ansible

## Prerequisites

- Set up your devops machine to run the deployment from (see details below).

- Set up your AWS account (see details below).

- Put your private key (as `default.pem`) for the servers into [.keys](./.keys) folder.

- Ensure that the private key permission is `600`.

- Adjust deployment variables if needed (e.g. region, availability zone, instance type, …) in the [variables](./variables) folder.


## Deploy the application

- Run `deploy` playbook to provision and set up the required services (e.g. EC2 instance) and install the application:

      ansible-playbook play/deploy.yml


## Remove the server

- Run `remove` playbook to de-provision the services:

      ansible-playbook play/remove.yml


## Start / stop the application

- Run `start` playbook:

      ansible-playbook play/start.yml

- Run `stop` playbook:

      ansible-playbook play/stop.yml


## Prerequisites description

### devops machine:

- Install [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

- Install [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

- authenticate in your AWS account (see below for permissions required) via AWS CLI.

If you use AWS SSO, add the profile to `~/.aws/config`:

    [profile devops]
    sso_session = ORG
    sso_account_id = <__ACCOUNT_ID__>
    sso_role_name = role-devops

    [sso-session ORG]
    sso_start_url = https://<___ID___>.awsapps.com/start/#
    sso_region = us-east-1
    sso_registration_scopes = sso:account:access

And then set the default profile:

    export AWS_DEFAULT_PROFILE=devops


### AWS Account:

- a `default` key to connect to the servers

- a `default` subnet (in the availability zone where the required instances are available)

-  an IAM role `geo-surreal-deploy` which will be assumed by a new EC2 instance to download container images from ECR:

    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ecr.GetDownloadUrlForLayer",
                    "ecr.BatchGetImage",
                    "ecr.GetAuthorizationToken"
                ],
                "Resource": [
                    "*"
                ]
            }
        ]
    }

- A devops user or a role executing the deploy scripts should have the following permissions:

    ```json
   {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:RunInstances",
                    "ec2:DescribeInstances",
                    "ec2:DescribeVpcs",
                    "ec2:DescribeSubnets",
                    "ec2:DescribeKeyPairs",
                    "ec2:DescribeAvailabilityZones",
                    "ec2:CreateSecurityGroup",
                    "ec2.AuthorizeSecurityGroupIngress",
                    "ec2:DescribeSecurityGroups",
                    "ec2:CreateTags",
                    "ec2:DescribeInstanceStatus",
                    "ec2:DescribeTags",
                    "ec2:DescribeInstanceAttribute",
                    "ec2:ModifyInstanceAttribute",
                    "ec2:ModifyInstanceMetadataOptions",
                    "ec2:TerminateInstances"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:GetInstanceProfile",
                    "iam:ListInstanceProfiles"
                ],
                "Resource": [
                    "*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "iam:GetRole",
                    "iam:PassRole"
                ],
                "Resource": [
                    "arn:aws:iam::<YOUR_AWS_ACCOUNT_ID>:role/geo-surreal-deploy"
                ]
            }
        ]
    }

Note: replace `<YOUR_AWS_ACCOUNT_ID>` with your real account id.
