# assignment-9
assignment

## Deployment Steps

### Part 1: S3 Setup (Static Assets)

1.  **Create S3 Bucket:**
    * Created an S3 bucket named `sda005-clarusway-assets` in the `eu-north-1` region. 
    * AWS Management Console: S3 -> Create bucket.

2.  **Upload Files:**
    * Uploaded the provided `index.html`, `logo.png`, and `sda.png` files to the created S3 bucket.
    * AWS Management Console: S3 -> your bucket -> Upload.

3.  **Configure S3:**
    * **Static Website Hosting:** Enabled static website hosting for the bucket, setting `index.html` as the index document.
        * AWS Management Console: S3 -> your bucket -> Properties -> Static website hosting -> Edit -> Enable.
    * **Bucket Policy:** Applied the following bucket policy to grant public read access to the objects in the bucket:

        ```json
        {
            "Version": "2012-10-17",
            "Statement": [{
                "Effect": "Allow",
                "Principal": "*",
                "Action": "s3:GetObject",
                "Resource": "arn:aws:s3:::sda005-clarusway-assets/*"
            }]
        }
        ```
        * AWS Management Console: S3 -> your bucket -> Permissions -> Bucket policy -> Edit.

### Part 2: Auto Scaling Group

1.  **Create Launch Template:**
    * Created a Launch Template to define the configuration for the EC2 instances in the ASG.
    * **AMI:** Used a standard Amazon Linux 2 AMI.
    * **Instance Type:** Selected a suitable instance type (e.g., `t2.micro`).
    * **Security Group:** Associated a security group that allows inbound HTTP traffic (port 80) from the ALB's security group.
    * **User Data Script:** Included the following script to install and configure NGINX and fetch the `index.html` file from S3 upon instance launch:

        ```bash
        #!/bin/bash
        yum update -y
        yum install nginx -y
        systemctl start nginx
        systemctl enable nginx
        aws s3 cp s3://sda005-clarusway-assets/index.html /usr/share/nginx/html/
        ```
        * AWS Management Console: EC2 -> Launch Templates -> Create launch template.

2.  **Configure ASG:**
    * Created an Auto Scaling Group using the created Launch Template.
    * **Minimum capacity:** 1
    * **Maximum capacity:** 3
    * **Desired capacity:** 2
    * **VPC and Subnets:** Selected the appropriate VPC and subnets across multiple Availability Zones for high availability.
    * **Health Checks:** Configured health checks to use both EC2 instance status checks and ELB health checks 
        * AWS Management Console: EC2 -> Auto Scaling Groups -> Create Auto Scaling group.

### Part 3: Application Load Balancer

1.  **Create Internet-facing ALB:**
    * Created an internet-facing Application Load Balancer.
    * **Scheme:** Internet-facing.
    * **VPC and Subnets:** Selected the same VPC and Availability Zones as the ASG.
    * **Security Group:** Created or selected a security group that allows inbound HTTP traffic (port 80) from `0.0.0.0/0`.
    * **Listeners:** Configured an HTTP listener on port 80.
    * **Target Group:** Created a new target group:
        * **Target type:** Instance
        * **Protocol:** HTTP
        * **Port:** 80
        * **Health checks:** Configured health checks with the `/` path.
    * Registered the Auto Scaling Group with the created target group.
        * AWS Management Console: EC2 -> Load Balancers -> Create Load Balancer -> Application Load Balancer.

2.  **Verify:**
    * **Access via ALB DNS name:** Accessed the website using the DNS name of the created ALB. The website should load, served by the NGINX instances.
    * **Round-robin traffic distribution:** Used the following command in the terminal to verify that the ALB distributes traffic across different instances:

        ```bash
        for i in 1..5; do curl -s http://lod-1005-948255613.eu-north-1.elb.amazonaws.com/ | grep "hostname"; done
        ```
## Success Criteria

1.  **Website accessible via both:**
    * S3 endpoint (e.g., `sda005-clarusway-assets.s3-website.eu-north-1.amazonaws.com`) displayed the static `index.html` page.
    * ALB endpoint displayed the `index.html` page served by the NGINX instances.
2.  **ASG automatically replaces terminated instances:** Verified by manually terminating an EC2 instance managed by the ASG, and observing that a new instance was automatically launched to maintain the desired capacity.
3.  **All assets load properly (HTML+logos):** The `index.html` page loaded correctly with the `logo.png` and `sda.png` images displayed.
