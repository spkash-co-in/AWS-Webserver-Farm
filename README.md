# AWS-Webserver-Farm

# Objective
Highly Available, load-balanced web server farm with mix of Linux-Apache & Windows-IIS instances deployed in multiple AZs on Amazon AWS

# Infrastructure as Code - AWS Cloud Formation
Simply put, AWS Cloud Formation enables us to define our infrastructure in JSON format.
Starting with enabling users to version their architecture defining Infrastructure as Code goes a long way in auditing and security of the organization. At any given time its only a few mouse clicks to trace changes and build an accurate overview.

# AWS Webserver Farm
Now we try to acheive the objective of this post by using a CloudFormation template.
We will go through the building blocks of the architecture and understand the pieces that help achieve our objective.

# Level One 
At the highest level one will find the following elements make up the CloudFormation template
This is the snapshot of the template at the highest level.

![hi-level](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/1.JPG)

1. Parameters
This section we provide the configurable parameters of our architecture
2. Mappings
This section I've left blank as I did not need any mappings. Ideally we select AZ mappings in this section.
3. Resources
This section defines and assembles the components that make up your architecture
4. Outputs
This section provides the out which in most cases is a URL. In our example the output equals that ELB URL that distributes traffic to our web server farm. An example from one of my runs is given below.

`http://elb-121503496.ap-southeast-1.elb.amazonaws.com/`

## Parameters
We separate out the configurable parameters of your architecture and provide in this section. It typically includes a list of options from which the user selects one. For example, you could provide the user with a selection of Linux AMI images to pick from to use as EC2 instance. In this example we have extracted out

![parameters](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/parameters.JPG)


### Keyname - Key pair that will be used for IAM credentials

![keyname](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/keyname.JPG)


### AMI and Instance Type 

1. LinuxAMI - A list of AMIs for our linux webserver farm 
2. WindowsAMI - A list of AMIs for our windows webserver farm 
3. InstanceType - Type of instances to use

![amitype](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/linuxami-windowsami-instancetype.JPG)


### NumberOfServers - Min and Max servers for the Autoscaling group

![numberofserver](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/numberofservers.JPG)





## Resources
This forms the meat of your defintion. We will drill down to the VPC level that will help us grasp the infrastructure conceptually. We are creating a VPC with two subnets - one for linux boxes and one for windows boxes. The snapshot below provides more details.




## VPC and Subnet

![vpc](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/vpc.JPG)

* VPC --> "CidrBlock": "172.31.0.0/16"
  * SubnetLinux
    * AZ --> "AvailabilityZone": "ap-southeast-1a"
    * CIDR  -> "CidrBlock": "172.31.38.0/24"
  * SubnetWindows
    * AZ --> "AvailabilityZone": "ap-southeast-1b"
    * CIDR --> "CidrBlock": "172.31.40.0/24"


![subnets](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/subnets.JPG)


    
## Security Groups
The next point of interest are the security groups which define rules on who can access what and which ports to open or close. In our example we are creating 

### LoadBalancerSecurityGroup
  * We allow port 80 incoming traffic to this group from everywhere ("CidrIp": "0.0.0.0/0")

![LB security group](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/loadbalancesecuirtygroup.JPG)

### LinuxSecurityGroup
  * We allow port SSH (22) for this group from everywhere
  * We allow HTTP (80) traffic only from LoadBalancerSecurityGroup

![linux security group](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/linuxsecuritygroup.JPG)

### WindowsSecurityGroup. 
  * We allow port RDP (3389) for this group from everywhere
  * We allow HTTP (80) traffic only from LoadBalancerSecurityGroup

![windwos security group](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/windowssecuritygroup.JPG)

  
## Launch Configuration
Next we look at the Launch Configurations. We have the following defined

### LinuxLaunchConfiguration
    * Image ID
    * Instance Type
    * Security Group
    * Key pair
    * Block Device Mappings
    * Metadata
    * UserData

![linux launch config](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/linuxlaunchconfiguration.JPG)


### WindowsLaunchConfiguration
    * Image ID
    * Instance Type
    * Security Group
    * Key pair
    * Block Device Mappings
    * Metadata
    * UserData

![windows launch config](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/windowslaunchconfiguration.JPG)



## LoadBalancer
Now we create the loadbalancer for our setup that listens to HTTP incoming requests and distributes traffic to both instances on both the subnets we have created. Not that crosszone is marked true for the configuration making it a CrossZone-ELB capable of distributing traffic across zones.


![loadbalancer](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/loadbalancer.JPG)

## LoadBalancer Health Checks
The following snapshots details that the Loadbalancer is configured to listen to traffic on HTTP port 80. The HealthChecks sections states which URL of the instances the Loadbalancer uses to check the health of the instance.

![loadbalancer healthcheck](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/loadbalancerhealthcheck.JPG)


## Autoscaling Group
Finally we configure the Autosacling group building on the LaunchConfigurations defined in the previous section. We create two Autoscaling Groups both joining the same LoadBalancer but starting in different subnets with different launch configurations.

### AutoscalingGroupLinux
  * Loadbalancer
  * LaunchConfiguration
  * VPCZoneIdentifier indicates the subnet
  * Max/Min/Desired capacity 

![linuxAS](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/linuxAS.JPG)


### AutoscalingGroupWindows
  * Loadbalancer
  * LaunchConfiguration
  * VPCZoneIdentifier indicates the subnet
  * Max/Min/Desired capacity 
  

![windowsAS](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/windowsAS.JPG)

## Block Device Mappings
This section details the block device mappings used in the launch configurations. 

![blockdevice](https://github.com/spkash-co-in/AWS-Webserver-Farm/blob/master/linuxblockdevicemappings.JPG)


## Windows Metadata
1. Start the IIS server
2. ebsConfig
  * Pull content from S3 and setup IIS content directory

## Linux Metadata

1. startHttp
  * Uses yum and installs httpd, then starts httpd
2. ebsConfig
  * We mount the EBS volume and create some content
  
  

## Windows UserData
Adds IIS Windows service and invokes cfn-init


## Linux UserData
Uses yum to install aws-cfn-bootstrap and init


# List of AWS services used

    * EC2
    * ELB
    * EBS volumes
    * S3 buckets
    * CloudFormation'
    * IAM
    * Security groups with detailed Ingress/Egress rules for HTTP + SSH + RDP traffic
    * Boxes accept HTTP requests only from ELB, not exposed to public
    * with NAT translation and IGW structure
    * CloudWatch
    * IGW
    * VPC, Subnet, NAT,  Network ACL, Route, RouteTable
    * Bootstrap Windows AMI 
    * Bootstrap Linux AMI

