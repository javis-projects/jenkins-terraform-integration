# jenkins-terraform-integration
Jenkins Terraform Integration | How do you integrate Terraform with Jenkins | Automate Infrastructure setup using Terraform and Jenkins | Remote Store in S3 Bucket

How to provision resources in AWS cloud using Terraform and Jenkins. We will also learn how to store terraform state info remotely in AWS S3 bucket.

![image](https://user-images.githubusercontent.com/104481671/200746110-17792f1e-8c8d-4424-ad0b-6fc897088a70.png)

1. We will create S3 bucket for storing terraform state info and Dynamo DB table for locking capability. 

2. We will try to create an EC2 instance and S3 Bucket using Terraform and Jenkins in AWS cloud. Look at the diagram that describes the whole flow. 

=======================================

Pre-requistes:
1. Create S3 bucket for storing TF state
2. Create dynamo DB table for providing lock capability
3. Jenkins is up and running
4. Terraform is installed in Jenkins
5. Terraform files already created in your SCM
6. Make sure you have necessary IAM role created with right policy and attached to Jenkins EC2 instance. see below for the steps to create IAM role.

-----------------------------------------------------

STEPS:

Step # 1 - 
Create S3 Bucket:
-----------------

1. Login to AWS, S3. Click on create S3 bucket.

2. Block all public access, enable bucket versioning as well.

3. Enable encryption.


Step # 2 - 
Create DynamoDB Table
---------------------

1. Create a new table with
1.1 Partition Key= LockID


Step # -3 
Create IAM role to provision EC2 instance in AWS 
------------------------------------------------

The IAM role should have the following 3 policies:
1. EC2AmazonEC2FullAccess 
2. S3 AmazonS3FullAccess, type 
3. DynamoDB FullAccess

Attach three policies


Step 4 - 
Assign IAM role to EC2 instance
-------------------------------

Go back to Jenkins EC2 instance, click on EC2 instance, Security, Modify IAM role


Step 5 - 
Create a new Jenkins Pipeline
-----------------------------

1. Go to Jenkins Server
2. Ensure you have the following installed;
    1. Terraform
    2. Git
    3. Terraform pligins
    4. Git credentials
  


3. Give a name to the pipeline you are creating.
- JenkinsTerraformInfraPipeline

Step 6 - 
Add parameters to the pipeline
-------------------------------

1. Click checkbox - This project is parameterized 

2. Add parameter = Choose Choice Parameter

3. Enter name as action

4. Choices: (it should be in two lines)
   Type apply and enter and 
   Type destroy
  
5. Description: Give a description

      Go to Pipeline section

6. Add below pipeline code and modify per your GitHub repo configuration.

pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
            checkout scm
            }
        }
        
        stage ("terraform init") {
            steps {
                sh ('terraform init -reconfigure') 
            }
        }
        stage ("terraform plan") {
            steps {
                sh ('terraform plan') 
            }
        }
                
        stage ("terraform Action") {
            steps {
                echo "Terraform action is --> ${action}"
                sh ('terraform ${action} --auto-approve') 
           }
        }
    }
}

-------------------

pipeline {
    agent any

    stages {
        stage('checkout') {
            steps {
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/javis-projects/jenkins-terraform-integration']]])
            }
        }
        stage ("init"){
            steps {
                sh ("terraform init")
            }
        }
        stage ("plan"){
            steps {
                sh ("terraform plan")
            }
        }
        stage ("action"){
            steps {
                echo "Terraform action is ---> ${action}"
                sh ("terraform ${action} --auto-approve")
            }
        }
    }
}

7. Click on Build with Parameters and choose apply to build the infrastructure or choose destroy if you like to destroy the infrastructure you have built. 

- Click on Build With Parameters,
- choose apply from the dropdown

   Now you should see the console output if you choose apply.

Step 7 - 
Login to AWS console
---------------------

- Login to S3 Bucket, you should see terraform state info is also added



9. How to Destroy all the resources created using Terraform?
   - run the Jenkins Pipeline with destroy option.



References: 
1. https://www.coachdevops.com/2021/12/jenkins-pipeline-terraform-integration.html
2. https://www.youtube.com/watch?v=hyP3RleaQ_o&t=18s

