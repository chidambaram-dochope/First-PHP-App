# First-PHP-App
this is my first PHP app that i am putting on GitHub
# Complete Guide: Setup CodePipeline for PHP Application Deployment on EC2

## What is CodePipeline?

CodePipeline is like an automated assembly line for your code. When you make changes to your PHP application, it automatically tests and deploys those changes to your server without you having to do it manually.

## Prerequisites (What You Need Before Starting)

### 1. Create an AWS Account

- Go to aws.amazon.com and sign up
- You'll need a credit card (AWS has a free tier for beginners)
- Complete the verification process

### 2. Create a GitHub Account

- Go to github.com and sign up with your email
- GitHub is like Google Drive but for code - it stores your application files

## Step 1: Prepare Your PHP Application on GitHub

### Understanding GitHub Basics

- **Repository (Repo)**: Think of it as a folder that contains all your application files
- **Commit**: Saving changes to your files
- **Push**: Uploading your changes to GitHub

### Create Your First Repository

1. Log into GitHub
2. Click the green "New" button or the "+" icon in the top right
3. Name your repository (example: "my-php-app")
4. Make it "Public" (free option)
5. Check "Add a README file"
6. Click "Create repository"

### Upload Your PHP Files

1. In your new repository, click "uploading an existing file"
2. Drag and drop your PHP files or click "choose your files"
3. Add a commit message like "Added my PHP application"
4. Click "Commit changes"

## Step 2: Create and Configure EC2 Instance

### Launch EC2 Instance

1. Go to AWS Console → EC2 service
2. Click "Launch Instance"
3. Choose "Amazon Linux 2" (free tier eligible)
4. Select "t2.micro" instance type (free tier)
5. Create a new key pair:
    - Name it something memorable like "my-php-key"
    - Download the .pem file and keep it safe
6. Configure Security Group:
    - Allow SSH (port 22) from your IP
    - Allow HTTP (port 80) from anywhere
    - Allow HTTPS (port 443) from anywhere
7. Launch the instance

### Install Required Software on EC2

1. Connect to your EC2 instance using SSH
2. Run these commands one by one:

```bash
# Update the system
sudo yum update -y

# Install Apache web server
sudo yum install httpd -y

# Install PHP
sudo yum install php php-mysql -y

# Install CodeDeploy agent
sudo yum install ruby wget -y
cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto

# Start Apache
sudo systemctl start httpd
sudo systemctl enable httpd
```

## Step 3: Create IAM Roles (Permission Settings)

### Create CodePipeline Service Role

1. Go to AWS Console → IAM service
2. Click "Roles" → "Create role"
3. Select "AWS service" → "CodePipeline"
4. Attach policy: "AWSCodePipelineFullAccess"
5. Name it "CodePipelineServiceRole"
6. Create the role

### Create CodeDeploy Service Role

1. Create another role
2. Select "AWS service" → "CodeDeploy"
3. Choose "CodeDeploy - EC2/On-premises"
4. The required policies will be attached automatically
5. Name it "CodeDeployServiceRole"
6. Create the role

### Create EC2 Instance Profile

1. Create another role
2. Select "AWS service" → "EC2"
3. Attach policies:
    - "AmazonS3ReadOnlyAccess"
    - "AWSCodeDeployRole"
4. Name it "EC2CodeDeployRole"
5. Create the role
6. Go to EC2 → Instances → Select your instance
7. Actions → Security → Modify IAM role
8. Attach the "EC2CodeDeployRole"

## Step 4: Create S3 Bucket (Storage for Your Code)

1. Go to AWS Console → S3 service
2. Click "Create bucket"
3. Name it uniquely (example: "my-php-app-deployment-bucket-12345")
4. Keep all default settings
5. Create the bucket

## Step 5: Set Up CodeDeploy Application

### Create CodeDeploy Application

1. Go to AWS Console → CodeDeploy service
2. Click "Create application"
3. Application name: "MyPHPApp"
4. Compute platform: "EC2/On-premises"
5. Create application

### Create Deployment Group

1. In your CodeDeploy application, click "Create deployment group"
2. Deployment group name: "MyPHPApp-DeploymentGroup"
3. Service role: Select "CodeDeployServiceRole" (created earlier)
4. Deployment type: "In-place"
5. Environment configuration: "Amazon EC2 instances"
6. Add tag: Key="Name", Value=(your EC2 instance name)
7. Install CodeDeploy Agent: "Now and schedule updates"
8. Load balancer: Uncheck "Enable load balancing"
9. Create deployment group

## Step 6: Prepare Your GitHub Repository for Deployment

### Add Required Files to Your Repository

You need to add these special files to tell CodeDeploy how to handle your application:

#### Create appspec.yml file

1. In your GitHub repository, click "Create new file"
2. Name it "appspec.yml"
3. Add this content:

```yaml
version: 0.0
os: linux
files:
  - source: /
    destination: /var/www/html
    overwrite: yes
hooks:
  BeforeInstall:
    - location: scripts/install_dependencies.sh
      timeout: 300
      runas: root
  ApplicationStart:
    - location: scripts/start_server.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: scripts/stop_server.sh
      timeout: 300
      runas: root
```

#### Create scripts folder and files

1. Create a new folder called "scripts"
2. Inside scripts folder, create three files:

**install_dependencies.sh:**

```bash
#!/bin/bash
yum update -y
yum install -y httpd php
```

**start_server.sh:**

```bash
#!/bin/bash
systemctl start httpd
systemctl enable httpd
```

**stop_server.sh:**

```bash
#!/bin/bash
systemctl stop httpd
```

3. Commit all these changes to your repository

## Step 7: Create CodePipeline

### Set Up the Pipeline

1. Go to AWS Console → CodePipeline service
2. Click "Create pipeline"
3. Pipeline name: "MyPHPApp-Pipeline"
4. Service role: "New service role" (or select existing "CodePipelineServiceRole")
5. Artifact store: "Default location" (uses S3)
6. Click "Next"

### Configure Source Stage

1. Source provider: "GitHub (Version 2)"
2. Click "Connect to GitHub"
3. Connection name: "MyGitHubConnection"
4. Click "Connect to GitHub" and authorize AWS
5. Repository name: Select your repository
6. Branch name: "main" (or "master")
7. Output artifacts: "SourceOutput"
8. Click "Next"

### Skip Build Stage

1. Click "Skip build stage" (since PHP doesn't need compilation)
2. Confirm by clicking "Skip"

### Configure Deploy Stage

1. Deploy provider: "AWS CodeDeploy"
2. Region: Select your region (same as EC2)
3. Application name: "MyPHPApp"
4. Deployment group: "MyPHPApp-DeploymentGroup"
5. Input artifacts: "SourceOutput"
6. Click "Next"

### Review and Create

1. Review all settings
2. Click "Create pipeline"

## Step 8: Test Your Pipeline

### Trigger Deployment

1. Your pipeline should start automatically
2. You can watch the progress in the CodePipeline console
3. The process goes: Source → Deploy
4. Each stage will show green when successful

### Verify Deployment

1. Go to your EC2 instance's public IP address in a browser
2. You should see your PHP application running
3. If you see the Apache default page, your files might not be in the right location

## Step 9: Make Changes and See Automatic Deployment

### Test Automatic Deployment

1. Go to your GitHub repository
2. Edit one of your PHP files
3. Make a small change (add a comment or change some text)
4. Commit the changes
5. Go back to CodePipeline - it should automatically start deploying your changes!

## Troubleshooting Common Issues

### Pipeline Fails at Deploy Stage

- Check that your EC2 instance has the CodeDeploy agent running
- Verify IAM roles are correctly attached
- Ensure your appspec.yml file is correctly formatted

### Can't Access Your Application

- Check EC2 security groups allow HTTP traffic
- Verify Apache is running: `sudo systemctl status httpd`
- Check file permissions in /var/www/html

### GitHub Connection Issues

- Make sure you've authorized AWS in your GitHub settings
- Try reconnecting to GitHub in CodePipeline settings

## What Happens Next?

Every time you push changes to your GitHub repository, CodePipeline will automatically:

1. Detect the changes
2. Download your updated code
3. Deploy it to your EC2 instance
4. Your website will be updated with zero downtime!

This is called "Continuous Deployment" - it saves you from manually uploading files every time you make changes.

## Security Best Practices

- Regularly update your EC2 instance
- Use strong passwords and keep your .pem file secure
- Monitor your AWS billing dashboard
- Set up CloudWatch alarms for unusual activity

## Cost Considerations

- EC2 t2.micro is free tier eligible for 12 months
- CodePipeline: First pipeline free, then $1/month per additional pipeline
- CodeDeploy: Free for EC2 deployments
- S3: Minimal storage costs for your code artifacts

You now have a fully automated deployment pipeline for your PHP application!
