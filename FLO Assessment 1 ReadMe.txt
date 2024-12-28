AWS Infrastructure and Twoge Application Deployment Guide
This guide walks you through deploying the Twoge application on AWS infrastructure, including VPC setup, S3 storage configuration, EC2 instance launch, application setup, auto-scaling, and SNS notifications.
1. Create an Amazon VPC with Two Public Subnets
First, you need to create a Virtual Private Cloud (VPC) with two public subnets in the us-west-2 region.
	• Go to the VPC Console to manage your VPC configuration.
	• Ensure that both subnets are public, meaning they have route tables configured to route traffic to the internet gateway.
2. Host Static Files on an S3 Bucket
Create an S3 bucket to store static files like images and videos used by the application.
	• Go to the S3 Console and create the bucket flo-exam-s3b.
	• Upload your static files to the bucket, e.g., images, videos, etc.
3. Create a Bucket Policy for Public Access
To allow public access to the static files in your S3 bucket, you need to apply a bucket policy that grants s3:GetObject permissions.
Example Bucket Policy (JSON):

json
Copy code
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::flo-exam-s3b/*"
        }
    ]
}
	• Apply this policy in the "Permissions" tab of your S3 bucket. This will make the static content publicly accessible.
	• Example URL for accessing a static file:

plaintext
Copy code
https://flo-exam-s3b.s3.us-west-2.amazonaws.com/static/img/twoge.png
4. Launch an EC2 Instance with Amazon Linux 2
Deploy an EC2 instance to host the Twoge application.
	• Go to the EC2 Console.
	• Select Amazon Linux 2 AMI for your instance, and configure the instance with the necessary settings like instance type, security group, key pair, and IAM roles.
	• Once the EC2 instance is launched, connect via SSH.
5. Install and Configure the Twoge Application on EC2
SSH into your EC2 instance and set up the Twoge application. Here is a sample script to install dependencies, clone the repository, and configure the app.
EC2 Setup Script:

bash
Copy code
# Update and install dependencies
sudo apt update
sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv git nginx -y
# Clone the Twoge repository
git clone https://github.com/chandradeoarya/twoge.git
cd twoge
# Create and activate a Python virtual environment
python3 -m venv twoge-env
source /home/ubuntu/twoge/twoge-env/bin/activate
# Install Python dependencies
pip install flask flask-migrate gunicorn
# Ensure no process is using port 8000
sudo fuser -k 8000/tcp
# Set up Gunicorn as a systemd service
echo "
[Unit]
Description=gunicorn daemon for Twoge app
After=network.target
[Service]
User=ubuntu
Group=ubuntu
WorkingDirectory=/home/ubuntu/twoge
ExecStart=/home/ubuntu/twoge/twoge-env/bin/gunicorn --workers 3 --bind 0.0.0.0:8000 app:app
[Install]
WantedBy=multi-user.target
" | sudo tee /etc/systemd/system/gunicorn.service
# Reload systemd, enable, and start Gunicorn service
sudo systemctl daemon-reload
sudo systemctl enable gunicorn
sudo systemctl start gunicorn
# Install and configure Nginx
sudo apt install nginx -y
# Restart Nginx to apply changes
sudo systemctl restart nginx
6. Create an Amazon ALB and Configure It to Route Traffic
	• Create an Application Load Balancer (ALB) to distribute traffic to your EC2 instances.
	• In the Listener configuration, add a rule to forward HTTP traffic (port 80) to your EC2 instances.
7. Create an Auto Scaling Group (ASG)
	• Configure an Auto Scaling Group (ASG) to automatically scale your EC2 instances based on traffic.
	• Ensure the ASG is associated with your ALB to handle the load balancing automatically.
	• Define scaling policies (e.g., scale out when CPU utilization exceeds 80%).
8. Use Amazon SNS for Notifications
	• Create an SNS topic to receive notifications when the number of EC2 instances in your ASG changes.
	• Set up email subscriptions to receive alerts about scaling actions, including when instances are launched or terminated.
	• Use the SNS Console to configure your notifications.
	• Ensure your EC2 Auto Scaling group is configured to send notifications to the SNS topic.
9. Stop EC2 Instance and Trigger SNS Notification
	• Stop an EC2 instance manually through the EC2 console.
	• SNS should send an email notification when the server is shut down or terminated.
10. Run Instance Stress Test and Observe Auto Scaling Behavior
Use a stress testing script to simulate heavy traffic to your EC2 instance, and observe the behavior of your Auto Scaling Group.
Stress Testing Script:

python
Copy code
# Simple Python stress test for the EC2 instance
import time
import random
while True:
    # Simulate load by creating random number calculations
    _ = random.random() ** random.random()
    time.sleep(0.1)
	• Run this script on the EC2 instance, and monitor the scaling of your ASG in response to the increased load.

Conclusion
By following the steps in this guide, you will have set up a highly scalable and resilient architecture using AWS services to deploy the Twoge application. The auto-scaling setup ensures that the application can handle increased traffic, and the SNS notifications keep you informed about scaling actions.
Let me know if you need any help with specific steps!
