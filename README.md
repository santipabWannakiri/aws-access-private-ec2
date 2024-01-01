# aws-access-private-ec2

## Introduction
Public IP addresses have long been a convenient means for direct access to EC2 instances. However, as of February 1, 2024, AWS has introduced charges for public IPv4 addresses. This means that even small POC instances with Auto-Assigned Public IPs will incur charges. In light of this, alternative methods for accessing EC2 instances without relying on public IPs are being explored.

Refer : [New – AWS Public IPv4 Address Charge + Public IP Insights](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)

## EC2 Instance without public IPv4 
<p align="center">
  <img src="images/private-subnet-ex.png" alt="image description" width="400" height="300">
</p>
Creating a subnet within a VPC and launching EC2 instances in that subnet is considered a best practice for enhancing security. This approach ensures that the EC2 instances do not directly receive public IPv4 addresses. This is particularly advantageous when the instances running the application are not directly involved with or connected to end-users. In such scenarios, providing public IPv4 addresses to these instances is unnecessary, aligning with security best practices.

Accessing instances without public IPv4 addresses in private subnets requires additional configurations before able to access to EC2 instant in private subnet.

## Connect to your instances without requiring a public IPv4
AWS provides various methods to securely access EC2 instances in private subnets. The methods you mentioned are commonly used for different use cases such as \
`AWS VPN` Creates an encrypted tunnel over the internet, allowing private communication between your on-premises network and instances in the VPC.\
`Bastion Hosts` A bastion host is deployed in a public subnet with a public IP. Users connect to the bastion host first and then jump to private instances.\
`AWS Systems Manager Session Manager` Allows you to access instances without the need for direct public IP addresses. Accessible through the AWS Management Console, AWS CLI, or SDK.

However, all of the way above, there are pros and cons to the differences in use cases. For example, Bastion Host still needs a public IP and also needs to dedicate an additional instant to be an intermedia to connect to your target instant in a private subnet.

Therefore, AWS launched a new feature called `EC2 Instance Connect Endpoint Service` that can help you connect to your instant without a public IP, and there is no additional cost charge.
<p align="center">
  <img src="images/ec2-connect-endpoint.png" alt="image description" width="500" height="400">
</p>

Refer : \
[Secure Connectivity from Public to Private: Introducing EC2 Instance Connect Endpoint](https://aws.amazon.com/blogs/compute/secure-connectivity-from-public-to-private-introducing-ec2-instance-connect-endpoint-june-13-2023/)

[Connect to your instances without requiring a public IPv4 address using EC2 Instance Connect Endpoint](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/connect-with-ec2-instance-connect-endpoint.html)

## Using the EC2 Instance Connect Endpoint Service

### IAM USER, Group, Policy and Role
<p align="center">
  <img src="images/user-group-policy-role.png" alt="image description" width="500" height="300">
</p>

Before we start to create any AWS resources and EC2 instants, I would like to start with access management. According to the illustration above, we're going to login to root user and configure IAM at the following:
1. Create `IAM USER`
* `azd-admin` Responsible for creating AWS resources for the POC.
* `azd-assume` Used to assume a role via the CLI for accessing EC2 instances.
2. Create `User groups`
* Create a user group named `azd-creator-group`.
* Attach appropriate policies to the group, such as IAMAccess, AmazonEC2FullAccess, AmazonVPCFullAccess, etc.
3. Create `Role`
* Create a role named `azd-assume-role`
* Configure Trust relationships to IAM USER `azd-assume`
#### Now we can switch to IAM USER azd-admin to conduct the policy and endpoint service part
<p align="center">
  <img src="images/poc-scenario.png" alt="image description" width="300" height="400">
</p>

1. Create VPC and Private subnet 
* Create VPC with only 1 availwith IPv4 CIDR `10.0.0.0/16`
* Create 2 private subnet with IPv4 CIDR `10.0.2.0/24` and `10.0.3.0/24` in same Availability Zones 

2. Create Endpoint Service
* Navigate to VPC > Endpoints > Create endpoint
* Provide endpoint name
* Choose the service category as an `EC2 Instance Connect Endpoint`
* Choose the VPC that we have created on step 1
* Choose Security groups
* Choose the APP private subnet in the picture above
* Click Create endpoint
* Once the endpoint is created, please check the console and copy the `Endpoint ID`

Refer : [Create an EC2 Instance Connect Endpoint](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/create-ec2-instance-connect-endpoints.html#create-eice)

4. Create Policies
* Create custom Policies named `enable-enpoint-policy`
* 


[Grant IAM permissions to use EC2 Instance Connect Endpoint](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/permissions-for-ec2-instance-connect-endpoint.html#iam-OpenTunnel)


[connect to an instance using the instance ID](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-connect-methods.html#connect-linux-inst-eic-cli-ssh)


[Allow users to use EC2 Instance Connect Endpoint to connect to instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/permissions-for-ec2-instance-connect-endpoint.html#iam-CreateInstanceConnectEndpoint)



