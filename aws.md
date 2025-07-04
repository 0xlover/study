# What do you sell?
"I make sure your business doesn’t crash and lose €100k/hour."
"I cut your server costs by 70% so you keep more profit."
"I automate scaling so you don’t need to hire 10 sysadmins."
# Who needs this?
You’re right—not every company needs a full-time cloud engineer. But here’s who pays:
1. Startups & Scale-Ups (Your Best Clients)
They: Have an app, zero in-house infra knowledge.
You: Set up AWS so they don’t burn cash on bad setups.
Example: A fintech startup needs PCI-compliant, auto-scaling Go API. You build it → charge €5k–20k.
2. Mid-Sized Companies (Stable Money)
They: Use AWS but waste €10k/month on unused resources.
You: Audit their setup, optimize costs → take €100/hr or % of savings.
3. Enterprise (Boring but Big Money)
They: Have legacy systems (on-premise crap) and need to move to cloud.
You: Migrate them → charge €80k+/year as a contractor. 
# AWS services
## Computing
EC2(Elastic Compute Cloud) are virtual servers in the cloud
Lambda provides serverless compute meaning it runs code without servers
ECS(Elastic Container Service)/EKS(Elastic Kubernetes Service) are Docker containers and Kubernetes clusters
### Notes
EC2 has various instance types such as t4g.nano which is ARM and cheap or p4d.24xlarge which isn't
Lambda can have api.something.extention/users do stuff without the API server having the code for it
You don't manage servers as AWS runs your code on demand not having to pay for idle time
Provisioning is manually setting up servers and resources(e.g. I need 2 EC2 instancies) and serverless elimiates it
ECS uses AWS proprietary orchestration instead of Kubernetes and can use either EC2 or Fargate launch types
## Databases
RDS(Relational Database Service) are SQL relational databases
Aurora is an high performance RDS alternative
DynamoDB are noSQL databases
### Notes
Aurora is limited to MySQL and Postgres but offers 5x performances and useful features although it costs 20% more(often worth)
Aurora uses it's own proprietary SQL
DynamoDB is neither Mongo nor Redis, but a proprietary AWS one
## Storage
S3(Simple Storage Service) is object storage for data
EBS(Elastic Block Store) are block storage for EC2
### Notes
S3 stores files and does not expand filesystems like EBS and alternatives to it are Google Cloud Storage and Azure Blob Storage
It can move data to Glacier storage class for archiving
You can host a static website on S3
EBS drives gets attached to EC2 that can be gp3(general purpose SSDs) or io1/2(high IOPS)
## Networking
VPC(Virtual Private Cloud) provides an isolated network for your resources
API Gateway exposes APIs and connects to Lambda
Cloudfront is a CDN for static content
### Notes
VPC will contain resources such as EC2 instancies
It is composed by subnets, route tables and security groups for this resources
## Messaging
SNS(Simple Notification Service) for pub/sub notifications(emails and SMS)
SQS(Simple Queue Service) is a queue system
SES(Simple Email Service) are SMTP servers
### Notes
DLQ(Dead Letter Queues) store failed messagges after visibilty timeout has expired
## DevOps
CodePipeline/Codebuild are for CI/CD
CloudFormation(AWS only) and Terraform are for IaC(Infrastructure as Code)
### Notes
IaC meaning you can have a file saying "I want this 3 EC2 and this one SNS from AWS" which is awesome
## Security
IAM(Identity and Access Management) manages permissions meaning "who can do what"
KMS(Key Management Service) menages encryption keys
Secrets Manager stores database credentials and API keys
## More
Route 53 for DNS
WAF(Web Application Firewall) is auto explicative
AWS Config, GuardDuty, Cost Explorer and Backup
Apply least privilage principile when using IAM
Always prefere managed services over the rest if control isn't a requirement
# Common errors that suck
VPC Peering, must create a full mesh
S3 Bucket Names are unique across all AWS accounts
Lambda Timeouts are 3s by default
RDS Deletions are not going to happen if not setting 0
# Possible questions
## Exam tricks
The "most cost effective solution" will likely be serverless(Lambda and DynamoDB over EC2)
The "minimal operation overhead solution" will likely be managed services(RDS over EC2+MySQL)
# Regions
AWS Regions Cheat Sheet
Code            Location		  Cool Feature
us-east-1	      N. Virginia		Most services, cheapest
eu-west-1	      Ireland			  GDPR-compliant
ap-northeast-1	Tokyo				  Low latency for Asia
me-south-1		  Bahrain	      Middle East Gaming hubs
# UI
Favorite a service by clicking the star
Applications group resources and metadata
Search can be used to access services and resources, find service features and get to specific docs
The CloudShell icon actually opens up a CloudShell which runs on Amazon Linux number 2 ans has the AWS CLI available in it
ARN(AWS Resource Names) like "arn:partition:service:region:account-id:resource-type/resource-id" identify a specific resource on AWS
# IAM
When you set permissions with IAM policies, grant only the permissions required to perform a task. You do this by defining the actions that can be taken on specific resources under specific conditions, also known as least-privilege permissions. You might start with broad permissions while you explore the permissions that are required for your workload or use case. As your use case matures, you can work to reduce the permissions that you grant to work toward least privilege.
An IAM instance is tied to the account and secure across al regions
Enties can be users(people that may require access to a given service, for example Bob wants to create, start and stop EC2), groups(which are logical groups of people like the development team or the human resources team) and finally roles which are generally used to give a service itself access to another service, e.g. EC2 containers might have read access to S3)
Policies are JSON files that can be attached to entities to allow or deny access to services 
An IAM user can't have more then one username and one password which is optional if you wanna create a user that's just gonna use generated access key with the AWS CLI for example
Roles do not use Access Keys
You can enable or disable access keys and create two for a user
# AWS accounts are desposable
albertochdev+AWSAccountN1@gmail.com is a unique email pointing to the original email address you can use for an account(e.g. +whatever@gmail.com)
self contained, secure by default(least privilage) and with a unique ID
## After creating a new AWS account
1. Enable MFA for the root user and add factors based on the account importance, also allow IMA users to access billing informations in the root account settings
2. Create a new administrator usage for yourself and get the Access Keys(via CloudShell or the Control Panel) and enable MFA or create a specialized user for yourself using least privilage and enable MFA
3. Create additional IAM users or do the same in the IAM Identity Center and enable MFA this may be the developer and production users
4. Do more with users, groups, roles and policies with IAM
5. Billing Preferences
6. Set budgets
# KMS(Key Management Service)
Can encrypt every object in an S3 bucket, EBS, RDS or Redshift database using a generated DEK
KMS Keys are protected by FIPS 140-2 "hardware security modules"(governament requirements for cryptographic modules)
Logs all these keys usage with CloudTrail
## CMK(Customer Master Key)s
Used to encrypy, decrypt and generate data keys and can be managed or not
Must set explicit permissions for key usage as in "least access control"
You pay 1$ a month for a customer managed CMK
CMKs should be rotated after some time
This generated "data keys"(KEK)  can later be used to encrypt large amount of data
## Envelop
KMS KEK(Key Encryption Key) are used to decrypt a given service object DEK(key associated with that specific object)
A request will be made to KMS from that service to unwrap or decrypt the DEK to have access to the plaintext key required to access the object
Permissions are handled on the CMK keys and the IAM users/roles
# The Global Infrastracture
AWS Regions(e.g. Ohio, Seoul and Tokyo) contain a whole deployment of AWS Infrastracture meaning most services will be available there allowing us to design a system to be global distaster proof because of how spread they are
AWS Edge Locations function as Local Distrubution points and are even more spreads then Regions(Netflix may use Edge Locations to store shows in multiple places to get closer to the customers in certain zones based on request)
You pick the regions meaning you have a certain political governance that applies in each, this is apparently really important
Regions are mapped with names like Asian Pacific Sidney is ap-southeast-2
There used to be https://infrastracture.aws
## AZ(Availabily Zone)
Failures are often localized, AZs provide isolated power, storage, computing and more in a region
Asian Pacific is split into ap-southeast-2a, ap-southeast-2b and ap-southeast-2c
If one zone fails for a power outage or a fucking explosion idk, the others will keep on working
AZs are not always a datacenter, they can be splitted in more
VPC is one of the services that is provided by every AZ
### Globally resilient services
Imagine a database with the data spread literally across all available regions
The only way this can truly fail, is if the world fails all at once
Examples of this are IAM(Users) and Route 53(DNS)
Finally if the Region fail the AZs will fail too
# VPC(Virtual Private Cloud)
VPCs are virtual networks in the cloud
You will also be connecting your on-premise infrastructure to it for an hybrid environment(even one with more then one cloud platform)
There will be one default VPC in your account in a region
Private and isolated by default meaning they cannot be interacted with from the public internet
One default VPC per region, but many custom ones requiring you to configure everything end-to-end
Most serious projects will use custom VPCs
Default VPCs come preconfigured with AWS IGW(Internet Gateway), SG(Security Group)s and more on your behalf(e.g. us-east-1 has the default VPC CIDR 173.31.0.0/16 devided in /20 for each available AZ)
It's possible to delete a default VPC(meaning there can be an account with no VPCs) and can easily be recreated
## Quoting
"I tend to keep default VPCs active in my regions because some AWS services expect them, but I don't use them for anything production related for they are too inflexible"
# EC2(Elastic Cloud Compute)
Consumption is based on running instances
Instance are an operating system with a certain selection of resources
They can be running, stopped or termiated and private by default(in a single VPC subnet by default)
You will need to modify the VPC to allow public access
EC2s are AZ resiliant because they are in one
## AMI(Amazon Machine Image)
EC2s run AMIs like Arch, FreeBSD, Windows Server or something specialized like a Plex running on Windows Server AMI
Permissions are attached to AMI images and can be either private and public(meaning that everyone can use it)
Major operating systems AMIs are public and can be found in the marketplace https://aws.amazon.com/marketplace
Every AMI has an owner(can decide if it's private or public and has implict permissions) that can give it explicit permissions
Again EC2 instances are pay per use, charging for the resources attached to the operating system and more(AMI based, like the Plex on Windows Server one)
## EBS(Elastic Block Store)
Network storage that can be made available to instances
Even if an instance is stopped, if an EBS storage is attached you will still pay for it and the same goes for more like static IP addresses
### Connecting to an EC2 instance
On Windows you can use RDP(Remote Desktop Protocol) defaulting on 3389 and on Linux and BSDs SSH as usual defaulting on 22
Standard public/private key pairs are used(can be easily uploaded from the AWS Console)
# RPO(Recovery Point Objective)
Let's say a company gets continuos data or provides a service daily, one day a server goes down(disaster) for some reason
How long is data loss tolerated?
The RPO dictates how often you will need to run full backups, if it's 6 hours, it has to be at minimum at 6 hours
You usually want backups to happen more often then that like 3 or 1 hours
Often you will need to work with someone to decide what RPO, others the decision may just be handed to you
# RTO(Recovery Time Objective)
Same scenario, a company service goes down, but this time it's not about data, but the service itself
How long a system can be down after a failure
Monitoring is extremely important, it will tell you when something is down so you can get to it as soon as possible
There is a lot of situations that can be predicted and decisions that should be made beforehand such as "the server is down and for whatever reason unusable, so where am I going to deploy on?" and "how quick will testing be after it gets back up?"
# S3(Simple Storage Service)
Globally resilient, public and very cheap
I should think about it as the default storage service in AWS even though it's not a file system or block storage meaning you won't be able to mount it
Delivers Objects(the files you own) and Buckets(containers for Objects)
## Objects
Basically files, they have a key(filenames) and a value like the file porn.jpg and it's image binary data
They also have a Version ID, Metadata, Access Control and Subresources
## Buckets
Created in a specific Region and never leave unless moved
The name is globally unique across all regions and accounts
Holds from zero to infinite amount of data
Can hold an infinite amount of Objects(each max 5TB)
It has no structure meaning everything is at the same level
You can create Objects named like paths(e.g. /porn/shut.mp4) and the AWS UI will present them as folders
Object names are 3-63 characters, lower case, no underscores, start with a letter or number and can't be formatted like IPs
There is soft a limit of 100 Buckets per account and an hard one you can push through using support requests of 1000
# Cloudformation(the AWS specific Terraform)
Create AWS infrastructure through template files
Can be YAML(YAML ain't Markup Language) or JSON(Javascript Object Notation)
Apply templates by uploading it from the CloudFormation UI using an S3 URL, a normal file upload, synching a Git repository or using the AWS CLI, it will then be available in an S3 Bucket and referred to as a Stack
## Categories
AWS Template Format Version, which is optional, if present the Description has to be right next to it
Description, the description of the template, needs to be right after
Parameters, allows you to provide options in the UI menu and in the CLI when applying a template(e.g. InstanceType, could be t3.micro, t2.micro or m4.large)
Mappings, key value pairs for lookup
Resources, like S3 buckets, EC2 instances and RDS databases
Outputs, what will be presented after the template is applied
There are also Rules, Conditions and Transform!
## Example
This is part of a template file that creates a simple Free Tier EC2 instance for later use with an attached SG allowing SSH and HTTP
```yaml
Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: "t2.micro"
      ImageId: !Ref LatestAmiId
      IamInstanceProfile: !Ref SessionManagerInstanceProfile
      SecurityGroups:
        - !Ref InstanceSecurityGroup
  InstanceSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: Enable SSH access via port 22 and 80
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: !Ref SSHandWebLocation
        - IpProtocol: tcp
          FromPort: '80'
          ToPort: '80'
          CidrIp: !Ref SSHandWebLocation
```
Instances have types like AWS::EC2::Instance
Even the UI is controllable via the templates
Uploaded templates are reffered as Stacks
## CLI
```bash
# using a file
aws cloudformation create-stack --stack-name "name" --template-body template.yml
# using S3
aws cloudformation create-stack --stack-name "name" --template-url https://learn-cantrill-labs.s3.amazonaws.com/awscoursedemos/0058-aws-simplecfn/ec2instance.yaml
```
# CloudWatch
Monitor various metrics(such as CPU usage and available disk space) in AWS services, even though as a public service you can use it outside of AWS too
Some metrics are gathered natively(you'll find them in the Console when creating an alarm) and some others requiring you to use CloudWatch Agent which will be also used to monitor metrics outside of AWS
CloudWatch Alarms will be triggered for a metric that's being monitored and CloudWatch Events can respond to these(like auto scaling EC2 or sending a notification with SNS)
CloudWatch Events can also run on a scheduled time and CloudWatch Logs will keep everything logged for me
A single measurement of a metric is called a datapoint(e.g. CPU usage was 98.3 at 11PM of Dec 25th 2025)
## Namespace
Containers used to separate data
AWS/EC2 is the namespace for all metric data for EC2, you can't use this style of naming(e.g. no AWS/GAME)
Datapoints also have dimentions attached(e.g. a EC2 instance CPU usage metric also have InstanceID and InstanceType)

# Subreddit advice
Look for it
# Mandatory practicies
Allow only a set of public IPs and a set of AMIs
Course and host a static page on AWS S3 or CloudFront, EC2 Minecraft server using Free Tier, API using RDS
# Standards and definitions
The NIST(National Instute of Standards and Technology) and RFCs(for internet related standards) are the central go to places
NIST publications https://www.nist.gov/publications
IETF RFC index https://datatracker.ietf.org
# CVE(Common Vulnarbilities and Exposures)
Find them here https://www.cve.org
# Types of Cloud
Multi is using multiple providers like AWS, Azure and GCP
Hybrid is using both the public(AWS) and private(AWS Outpost) components of generally a single provider
Using an Hypervisor(HyperV, Proxmox and Unraid) in your infrastracture is very different compared to using a Private Cloud(AWS Outpost, Azure Stack and Google Anthos) from a provider
With true Private Cloud you will be using the same tooling to interact with both the Public and Private part(e.g. aws-cli)
## Quoting
"I do not recommend a third party tool that abstracts away for multi cloud scenarios"
# Models
IaaS(Infrastracture as a Service) like AWS EC2, you got the server and manage it
PaaS(Platform as a Service) like Vercel, you got the application and deploy it there
SaaS(Software as a Service) like AppleTV+, you pay for the application to use it

# High Availabilty
Designed to ensure working operations almost all the time meaning downtime is allowed
Maximizing online time not guaranteeing it
Usually a percentage(99.9% HA is online for a whole year -8h, 99.999% is the same meaning -6m)
Two servers one active one standby so that one can replace the other if it goes down
Copies will be used
# Fault Tolerance
Often mixed up with HA, but FT garantees a functionig system even during fautls(outages)
Even if a system has faults(one or more) it should continue to function while the faults are present and being fixed
HA isn't operating through failure, FT is
Obviously FT will be more expensive then HA
Example of an FT system would be a plane(has copies of components so that it can be repaired while still going, can't stop a plane mid-air to repair the engine) and AWS has to be treated the same
# Disaster Recovery
Planning about what to do when a disaster happens?
When HA or FT don't work, just fucking what then?
Devided into pre planning and DR process
During a disaster people are sleepless and shocked which isn't ideal to improvise(meaning a good DR process has to be in place)
Example, backups are offsite and keys should be available no matter what
# Route 53
It's AWS DNS register and hosts zones
Global resilient and globally available(no region selection)
1. Gets the domain
2. Creates a zonefile
3. Allocates 4 nameservers for your zone
Communicates with registries like PIR(Public Interest Registry) for a TLD such as .org
By delegating the nameservers, the registry indicates that those nameservers are authoritative for the specific domain
Can be used within a VPC
# IAM
## Policies
Used with a user, group or a role
Composed by one or more statements which grant or deny permissions to AWS Services to Resources(like * and arn:aws:s3:::furry)
### Examples
The user, group or role with this policies will have full access to every s3 bucket except for cat-pics!
```json
{
   "Version": "2012-10-17",
   "Statement": [
    {
       "Sid": "FullAccess",
       "Effect": "Allow",
       "Action": "s3:*",
       "Resource": ["*"]
    },
    {
       "Sid": "DenyCatBucket",
       "Effect": "Deny",
       "Action": "s3:*",
       "Resource": ["arn:aws:s3:::cat-pics", "arn:aws:s3:::cat-pics/*"]
    },
   ]
}
```
The 1st statement called "FullAccess" allows every action for the S3 service for every S3 bucket, but the 2nd denies those two specific ARNs!
The user, group or role won't be able to access the cat-pics bucket nor any object in it.
Every interaction with AWS is composed by the resource you are interacting with and the action on that resource.
### Overlapping
> DENY, ALLOW, DEFAULT DENY
1. Explicit denies just does that and nothing can override it.
2. Explicit allows take effect unless there's an explicit deny!
3. By default everything is denied.
Sally might be a user with policies and inside a group with policies trying to access a resource with policies.
Now guess what? At any point there might be an explit deny that stops her from accessing the resource :3
### Applying
Let's say we need to provide policies for a secret projects
#### Inline policies
Applying the policies to each account
#### Managed policies
Creates a policy and then apply it to each account
Usually for exceptional allows or denies
Will provide low management overhead compared to inline policies
## Users
Used for everything requiring long term access
### Quoting 
"If you can picture one thing like a human needing to access a service or an application that needs to do the same, 99% of the time you are gonna use an IAM user, remember that"
There's a limit of 5000 users per account and each one can be in 10 groups
For internet scale applications or company merges you'll be using IAM roles or Federation
Exam questions could play on this and would be probably wrong!

# ARNs
ARN(AWS Resource Names) like "arn:partition:service:region:account-id:resource-type/resource-id" identify a specific resource on AWS
## Format
arn:partition:service:region:account-id:resource-id
arn:partition:service:region:account-id:resource-type/resource-id
arn:partition:service:region:account-id:resource-type:resource-id
Should I use ":" or "/"? It's server convention based, but they can be used interchangeably(aws docs specify)
## Services
### IAM
user arn:aws:iam::account-id:user/user-name
policy arn:aws:iam::aws:policy/PolicyName like arn:aws:iam::aws:policy/AdministratorAccess
the account is skipped there meaning "aws" instead of "user" because it's global
### S3
bucket arn:aws:s3:::bucket-name
object arn:aws:s3:::bucket-name/key-name
### EC2
instance arn:aws:ec2:region:account-id:instance/instance-id
securty group arn:aws:ec2:region:account-id:security-group/security-group-id
### Lambda
function arn:aws:lambda:region:account-id:function:function-name
### More
arn:aws:sns:region:account-id:topic-name
arn:aws:sqs:region:account-id:queue-name
arn:aws:dynamodb:region:account-id:table/table-name
arn:aws:kms:region:account-id:key/key-id
arn:aws:rds:region:account-id:db:db-instance-name
arn:aws:logs:region:account-id:log-group:log-group-name
## Exceptions
Wildcards(e.g., arn:aws:s3:::bucket-name/*)
Paths(e.g., arn:aws:iam::account-id:role/path/role-name)
# Partitions
AWS partitions are isolated segments of the AWS global infrastructure, designed to meet regulatory, compliance, and data sovereignty requirements.
S3 for example would have three different deployments with API endpoints that may slightly differ in these three partitions
## Examples
Partition  Name                 Purpose
Standard	 arn:aws:...	        Default partition for global AWS services (public regions).
GovCloud	 arn:aws-us-gov:...	  For U.S. government workloads (FedRAMP, ITAR compliance).
China	     arn:aws-cn:...	      Operated by Sinnet (Beijing) and NWCD (Ningxia), compliant with Chinese laws.
## AWS Infrastructure Hierarchy
So Partitions(e.g. aws and aws-cn) contains Regions(us-east-1 and ap-northeast-1), now these are separated into AZs(us-east-1a and ap-southeast-2c) and finally there are also Edge Locations which are deployed globally to accelerate content delivery and comply with regional data law
# CLI
## Structure
```
aws <service> <operation> [parameters] 'options'
```
## Optionsc
--profile(Use different configured credentials)
--region(Specify AWS region)
--output(Change output format, like text, json and table)
--query(Filter output using JMESPath)
Look at that JSON query!
```bash
aws cloudfront list-distributions --query 'DistributionList.Items[].DomainName'
```

AWS simulations
AWS labs 