## Cafe Store:

A small café business that sells desserts and coffee.
The café has a single location in a large city.
They mostly gain new customers when someone walks by, notices the café, and
decides to try it.
I suggested that they should expand community awareness of what the café has to
offer. (So we started simple) 

## 1st Use case: The client suggested a business request to Launch a static website for the cafe
The client wanted the café to have a website that will visually showcase the
café's offerings.
It would also provide customers with business details, such as the **location of
the store,
** **business hours,** and **telephone number.** So I suggested to build a
static website based on the client use case.

**Task 1:** To Design a webpage using **html syntax and css**

**Task 2:** To Design an **S3** bucket to host the static website.

**Approach1:** Created the bucket in the **AWS Region** that is closest to the
people who are most likely to access it.

**Approach2:** Disabled Block all public access since it will be publicly
accessed.

**Approach3:** Enabled **Static website hosting** on the bucket.

**Task 3:** Uploading content to the **S3** bucket.

**Approach:** Uploaded the ```index.html``` file, the ``css`` and ``images`` 
folders to the **S3** bucket.

**Task 4:** Creating a bucket policy to grant public read access.
**Approach:** Created a bucket policy that grants **read-only** permission to
public anonymous users by using the **Bucket Policy** editor.

## 2nd Use Case: Based on the business requirement: Protecting website data
To implement a strategy to prevent the accidental overwrite and deletion of
website objects.
Due to making some changes to the website, I decided that this would be a good
time to explore object versioning.

**Task 5:** **Enabling versioning** on the **S3** bucket
**Approach:** In the **S3** console, **enabled versioning** on the **S3** bucket.

## 3rd Use Case: Based on the business requirement: Optimizing costs of S3 object storage
Now that I enabled versioning, I realized that the size of the S3 bucket will
continue to grow as I upload new objects and versions. To save costs, I decided
to implement a strategy to retire some of those older versions.

**Task 6:** Setting lifecycle policies

**Approach 1:** Designed a **lifecycle policy** to automatically move older
versions of the objects in the source bucket to S3 **Standard-Infrequent Access**
(S3 Standard-IA). The policy should also eventually expire the objects.

**Approach 2:** I Configured two rules in the website bucket's lifecycle
configuration.
In one rule, to move previous versions of all source bucket objects to
**S3 Standard-IA** after 30 days.
In the other rule, delete previous versions of the objects after 365 days.

## 4th Use Case : Based on the business requirement: Enhancing durability and planning for DR(Disaster Recovery)
Cross-Region replication is another feature of Amazon S3 that we can also use
to back up and archive critical data.

**Task 7:** Enabling cross-Region replication

**Approach:** In a different Region than the source bucket, created a second
bucket and enabled versioning on it.
The second bucket is the destination bucket.
On the source S3 bucket, enabled cross-Region replication. When creating the
replication rule, I made sure that I:

Replicated the entire source bucket.
Used the CafeRole for the **AWS Identity and Access Management (IAM) role**.
This **IAM** role gives Amazon **S3** the permissions to read objects from the
**source bucket** and replicate them to the **destination bucket**.

## 5th Use Case: Creating a Dynamic Website for the Café
After the café launched the first version of their website, However, customers
often asked whether they could place online orders.

**Approach:** Designed an EC2 instance to host a website and created a database.

## New business requirement: Creating development and production websites in different AWS Regions

**Task 8:** Creating an **AMI** and launching another **EC2** instance

## New business requirement: To Design a Scalable and highly available environment for the cafe
They want to ensure that their customers have a great experience when they visit
the website, and that they don’t experience any issues, such as lags or delays
in placing orders.
To ensure this experience, the website must be responsive, able to scale both
up and down to meet fluctuating customer demand, and be **highly available.**
Instead of overloading a single server, the architecture must distribute
customer order requests across multiple application servers so it can handle
the increase in demand.

To Implemente a **scalable and highly available environment**
Before changing the café’s application architecture, I had to evaluate its
current state.

**Task 1:** Inspecting the environment
This involve checking how the network is set up.

**Task 2:** Creating a **NAT gateway** for the second Availability Zone
To achieve high availability, the architecture must span at least two
**Availability Zones**. However, before I launch **Amazon Elastic Compute Cloud**
(Amazon EC2) instances for the web application servers in the second
**Availability Zone**, I created a **NAT gateway** for them. A **NAT gateway**
will allow instances that do not have a **public IP** address to access the internet.

**Approach:** Created a **NAT gateway** in the **Public Subnet** in the second
**Availability Zone**.
Configured the network to send **internet-bound traffic** from instances in 
**Private Subnet 2** to the **NAT gateway** I created.

**Task 3:** Creating a bastion host instance in a **public subnet**

**Approach:** Created a bastion host in a public subnet.

**Task 4:** Creating a launch template
An **Amazon Machine Image (AMI)** was created from the **CafeWebAppServer** instance.
In this task, I created a launch template by using this **AMI**.

**Task 5:** Creating an **Auto Scaling group**
Now that the launch template is defined, I created an Auto Scaling group for the instances.

**Approach:** Created a new **Auto Scaling Group** that met the following criteria:
**Launch template:** Uses the launch template that I created in the previous task 
**VPC:** Uses the **VPC** that was configured for this project
**Subnets:** Uses **Private Subnet 1** and **Private Subnet 2 **
Has a Group size configured as:
**Desired capacity:** 2
**Minimum capacity:** 2
**Maximum capacity:** 6
**Enables the **Target tracking scaling policy** configured as:
**Metric type:** **Average CPU utilization**
**Target Value:** 25
**Instances need:** 60 

**Task 6:** Creating a load balancer
Now that the web application server instances are deployed in private subnets,
I needed a way for the outside world to connect to them. In this task,
created a **load balancer** to distribute traffic across the private instances.
Created an **HTTP Application Load Balancer** that met the following criteria:
**VPC:** Uses the VPC configured
**Subnets:** Uses the two public subnets
Skips the **HTTPS security configuration settings**
**Security group:** Creates a new security group that allows HTTP traffic from
anywhere
**Target group:** Creates a new target group
Skips registering targets
Modified the **Auto Scaling group** that I created in the previous task by
adding this new **load balancer**. 

**Task 8:** Testing automatic scaling under load
In this task, I carried out a test to verify if the café application scales out automatically.
By using **Secure Shell (SSH)** passthrough through the bastion host instance,
used SSH to connect to one of the running web server instances.
**Note:** I will need to modify the **CafeSG security group** to **allow SSH traffic** over **port 22** from the **bastion host**.
From the web server instance, I performed a stress test. This test increases the load on the web server CPU:  ```sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm  ```
``sudo yum install stress -y stress --cpu 1 --timeout 600 ``

I verified that the **Auto Scaling group** deploys new instances.
By observing the **Amazon EC2 console**.
During the test, I observed that more web server instances are deployed.

## Update from the café
After implementing the client system requirements when traffic increases, the café application now successfully scales out.
The cafe could rely on automatic scaling to help them take orders and delight new customers.

Things I would have loved to implement:
A monitoring system like **datadog agent** to monitor the entire **AWS Cloud stack** deployed.
