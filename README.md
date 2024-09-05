# Flaskapp Deployment using Github Action Workflow

This repository demonstrates how to implement a CI/CD pipeline for a Python Flask application using GitHub Actions. The pipeline automates testing, building, and deploying the Flask app to both staging and production environments.

## Objective 

The objective of this project is to automate the CI/CD pipeline for a Python Flask application using GitHub Actions.

## Project Execution Steps:

**Step 1: Setup**
* Fork the python flask app repository
* Ensure the repository has two main branches:
    * main: For production.
    * staging: For testing and staging environments.

**Step 2: AWS EC2 Setup**

1. Launch an EC2 instance
  * Use an Ubuntu AMI
  * Assign Security Group:
    Ensure that the security group associated with the EC2 instance allows inbound traffic on:

    * SSH (port 22)
    * HTTP (port 80)
  * Ensure Public IP:
    Make sure your EC2 instance is assigned a public IP so it is accessible over the internet.

    
**Step 3. Set Up GitHub Secrets**
To securely connect GitHub Actions to your EC2 instance, you’ll need to add a few secrets to your GitHub repository. Follow these steps:

1.Go to your repository on GitHub.\
2.Navigate to Settings > Secrets and variables > Actions > New repository secret.\
3.Add the following secrets:

       SSH_PRIVATE_KEY: The private key for accessing your EC2 instance.
       SSH_HOST: The public IP of your staging EC2 instance.
       USER_NAME: The username for SSH access.
       HOSTNAME_PRODUCTION:The public IP of your production instance
   ![secrets](https://github.com/user-attachments/assets/bed9b00d-6709-4796-9667-59a4b4661689)

       

**Step 4. Configuring GitHub Actions**

 1. Create a Workflow File:

    Inside your repository, create the following directory structure:
    
  ``` 
   .github/   
    └── workflows/
        └── deploy.yml
 ```
 2. Define the GitHub Actions Workflow:
    
***Deployment to Staging Server***
```yaml
name: Python package

on:
  push:
    branches:
      - "staging"

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        # This is the version of the action for setting up Python, not the Python version.
        uses: actions/setup-python@v5
        with:
          # Semantic version range syntax or exact version of a Python version
          python-version: '3.x'
      # Install the dependencies 
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          cd flaskapp
          pip install -r requirements.txt
      - name: Test with pytest
        run: |
          pip install pytest
          cd flaskapp
          pytest test_app.py
      - name: Copy flaskapp into EC2 instance
        env:
           PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
           HOSTNAME: ${{ secrets.SSH_HOST }}
           USER_NAME: ${{ secrets.USER_NAME }}
           DESTINATION_PATH: "/home/ubuntu"
        run: |
          echo "$PRIVATE_KEY" > private_key && /bin/chmod 600 private_key
          /bin/scp -o StrictHostKeyChecking=no -i private_key -r ./flaskapp ${USER_NAME}@${HOSTNAME}:${DESTINATION_PATH}
          ssh -o StrictHostKeyChecking=no -i private_key ${USER_NAME}@${HOSTNAME} '
          sudo apt-get update -y &&
          sudo apt install python3 -y &&
          sudo apt-get install python3-pip -y &&
          cd flaskapp
          pip install -r requirements.txt --break-system-packages
          python3 app.py &
          '
```

***Deployment to Production***
```yaml
name: Deploy to Production

on:
  release:
    types: 
      - published
      
jobs:
  build:

    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
      - name: Set up Python
        # This is the version of the action for setting up Python, not the Python version.
        uses: actions/setup-python@v5
        with:
          # Semantic version range syntax or exact version of a Python version
          python-version: '3.x'
      # You can test your matrix by printing the current Python version
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          cd flaskapp
          pip install -r requirements.txt
      - name: Test with pytest
        run: |
          pip install pytest
          cd flaskapp
          pytest test_app.py
      - name: Copy flaskapp into EC2 instance
        env:
           PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
           HOSTNAME_PRODUCTION: ${{secrets.SSH_HOST}}
           USER_NAME: ${{secrets.USER_NAME}}
           DESTINATION_PATH: "/home/ubuntu"
        run: |
          echo "$PRIVATE_KEY" > private_key && /bin/chmod 600 private_key
          /bin/scp -o StrictHostKeyChecking=no -i private_key -r ./flaskapp ${USER_NAME}@${HOSTNAME_PRODUCTION}:${DESTINATION_PATH}
          ssh -o StrictHostKeyChecking=no -i private_key ${USER_NAME}@${HOSTNAME_PRODUCTION} '
          sudo apt-get update -y &&
          sudo apt install python3 -y &&
          sudo apt-get install python3-pip -y &&
          cd flaskapp
          pip install -r requirements.txt --break-system-packages
          python3 app.py &
          '
```

**Step 5: Push to GitHub and Trigger the Workflow**

 1.***Deploy to Staging:***

   * Push changes to the staging branch of your repository. This will trigger the GitHub Actions workflow to deploy the application to the staging environment.
     ![Screenshot 2024-08-27 180103](https://github.com/user-attachments/assets/3b1d4a39-68e9-4f3d-a897-52efe9eedeb3)

 2. ***Deploy to Production:***

   * Create a new release by going to the Releases section in your GitHub repository.
   * Click Draft a new release and create a new release tag (e.g., v0.0.7). This will trigger the GitHub Actions workflow to deploy the application to the 
   production environment.
![Screenshot 2024-09-04 005818](https://github.com/user-attachments/assets/13c9e4e0-bb04-4d9b-a941-a584e39882f8)


**Step 6: Access the Application:**

After a successful deployment, you can access your Flask application via the EC2 instance's public IP and port 8000:

  ` http://your-ec2-ip:8000`

            




     
        



