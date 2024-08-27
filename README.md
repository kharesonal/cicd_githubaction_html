# CI/CD Pipeline for HTML Deployment to EC2 Using GitHub Actions
This repository demonstrates how to set up a Continuous Integration/Continuous Deployment (CI/CD) pipeline using GitHub Actions to automatically deploy an HTML file to an Amazon EC2 instance running Nginx.

# Overview
This project sets up an automated deployment pipeline that triggers whenever changes are pushed to the main branch of this repository. The pipeline involves:

1.Checking out the code.

2.SSHing into an EC2 instance.

3.Preparing the target directory on the EC2 instance.

4.Copying the HTML code to the EC2 instance.

5.Restarting Nginx to serve the updated content.

# Steps
## Step 1: Create the Repository
Create a new repository on GitHub named cicd_githubaction_html

## Step 2: Create a Simple HTML File
Create a file named index.html in the repository

## Step 3: Create an EC2 Instance and Install Nginx
Launch a new EC2 instance and SSH into it. Once connected, install Nginx using the following commands:



```sudo apt update

sudo apt install nginx -y

sudo systemctl start nginx

sudo systemctl enable nginx
```


## Step 4: Set Up GitHub Actions Workflow
Create a new file in the repository at '.github/workflows/deploy.yml'

## Step 5: Update Secret Variables

In your GitHub repository, navigate to Settings -> Secrets and variables -> Actions -> New repository secret. 

Add the following secrets:

**EC2_HOST**: Public IP address of your EC2 instance.

**EC2_SSH_KEY**: Content of your private SSH key.

**EC2_USERNAME**: Username for SSH (usually ubuntu for Ubuntu instances).

## Step 6: Commit Changes

Make sure all changes are committed and pushed to the main branch.

## Step 7: Run the Workflow

Navigate to the Actions tab in your repository. You should see the workflow running automatically, deploying the HTML file to your EC2 instance, and restarting Nginx.

## Step 8: Access Your HTML File

Open a web browser and navigate to the public IP address of your EC2 instance to view the deployed HTML file.
