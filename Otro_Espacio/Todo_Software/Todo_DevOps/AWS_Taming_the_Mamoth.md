# AWS Taming the Mamoth

## General
![aws-app-mgmt-solutions.png](https://trello-attachments.s3.amazonaws.com/52ed247ef106ad5c2f83f91c/591330e74287f5f5e6b1428c/12546187530a1060f3aec1490c25608c/aws-app-mgmt-solutions.png)



## EC2
- Instance reboot: [1 - Stack Overflow](http://stackoverflow.com/questions/637790/what-happens-when-i-reboot-an-ec2-instance), [2 - AWS Docs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-reboot.html)
- IP pública vs IP privada. AWS security groups


## AMI
- Copy from Region to Region: [AWS Blog](https://aws.amazon.com/es/blogs/aws/ec2-ami-copy-between-regions/)


## RDS
- Creating a Read Replica
- What is Multi AZ for?


## EC2 - ELB
- Diferencias entre el ELB Classic y Application
- Configuración básica de uno Classic


## Code Deploy
- Basic configuration with EC2 instances
- Where to find deployment logs in machines
- Saving STDOUT for every bash command to a file in the server the deployment is happening

**Enabling load balancer in Code Deploy application ended in a very, very slow deployment**
Using [AWS provided scripts](https://github.com/awslabs/aws-codedeploy-samples/tree/master/load-balancing/elb#important-notice-about-handling-autoscaling-processes) for handling *auto scaling groups* processes and prevent load balancer from sending traffic to instances receiving a deployment, turned deployments time to be very slow per machine. See image below. *note scripts are more than a year old.*

![slow-deployment.png](https://trello-attachments.s3.amazonaws.com/52ed247ef106ad5c2f83f91c/591330e74287f5f5e6b1428c/5b6522dffed2770a93eb05f47f92e36c/slow-deployment.png)


After getting this error:

    [stderr]Waiting for instance to register to its load balancers
    [stderr]Checking 11 times, every 3 seconds, for instance i-05afac56e77611941 to be in state InService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Checking status of instance 'i-05afac56e77611941' in load balancer 'bucket-api-load-balancer-01'
    [stderr]Instance is currently in state: OutOfService
    [stderr]Instance failed to reach state, InService within 33 seconds
    \[stderr\][FATAL] Failed waiting for i-05afac56e77611941 to return to bucket-api-load-balancer-01

I enabled the load balancer for the Code Deploy app. Then, got this other error message:

    Role does not have correct permissions. role arn:aws:iam::832109556513:role/code_deploy sessionName d3TYE99IAO. for activityId="5" of activityType={Name: ExecuteCentralizedCommandOnInstanceActivity.runCentralizedCommand,Version: 1.00}

And to solve it added the `elasticloadbalancing:*` action to a policy belonging to `code_deploy` role. This is the policy:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "autoscaling:PutLifecycleHook",
                    "autoscaling:DeleteLifecycleHook",
                    "autoscaling:DescribeLifecycleHooks",
                    "autoscaling:RecordLifecycleActionHeartbeat",
                    "autoscaling:CompleteLifecycleAction",
                    "autoscaling:DescribeAutoscalingGroups",
                    "autoscaling:PutInstanceInStandby",
                    "autoscaling:PutInstanceInService",
                    "ec2:Describe*",
                    "elasticloadbalancing:*"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
    }

But it made the deployments to run really, really slow.


## EC2 - Auto Scaling/Launch Configurations
- cloud-init scripts are run as `root`: [AWS Docs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/user-data.html?shortFooter=false#user-data-shell-scripts)
- run script as non-root user: [Ubuntu forums](https://ubuntuforums.org/showthread.php?t=1684800&s=1ba8c8fa219433e49af516f0983b8ce7&p=10444586#post10444586)


## IAM: users, groups and roles (links en Raindrop)
- Roles, grupos y users
- [IAM identifiers](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_identifiers.html#Identifiers_ARNs)
- Let groups assume roles
- [Trust relationship](http://docs.aws.amazon.com/mobile-hub/latest/developerguide/service-role-trust-relationship.html)
- [elements of a policy](http://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements.html?shortFooter=true)
- best practices (security)
- Las diferencias entre roles y grupos (API vs keys)


## S3
- Mover archivos de server a aws s3: [s3cmd](https://medium.com/@darilldrems/how-to-transfer-large-files-to-amazon-s3-ae612b68a8c7#.w3ipqfvkh)


## Security
- [AWS security recomendations](https://www.airpair.com/aws/posts/building-a-scalable-web-app-on-amazon-web-services-p1#4-7-security)
- [AWS Security PDF](http://media.amazonwebservices.com/AWS_Security_Best_Practices.pdf)
- [OWASP](http://owasptop10.googlecode.com/files/OWASP%20Top%2010%20-%202013.pdf)
- [PCI](https://www.pcisecuritystandards.org/security_standards/documents.php?agreements=pcidss&association=pcidss)

