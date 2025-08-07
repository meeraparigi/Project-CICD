pipeline {
  agent any

  environment {
    AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
    TF_VAR_aws_region = "us-east-1"
    TF_VAR_bucket_name = "project-cicd-my-demo-s3-bucket"
  }

  stages {
    stage('Checkout'){
      steps {
         git branch: 'main', url: 'https://github.com/meeraparigi/Project-CICD.git'  
      }
    }

    stage('Initialize Terraform'){
        dir('terraform'){
            steps {
               sh 'terraform init'
            }
        }
    }

    stage('Validate Terraform Code'){
        dir('terraform'){
            steps {
               sh 'terraform validate'
            }
        }
    }
    
    stage('Create Terraform Plan'){
        dir('terraform'){
            steps {
               sh 'terraform plan'
            }
        }
    }

    stage('Create Inforastructure with Terraform'){
        dir('terraform'){
            steps {
               sh 'terraform apply -auto-approve'
            }
        }
    }
  }
}
