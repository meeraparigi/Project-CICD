pipeline {
  agent any

  environment {
    AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
    AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
  }

  stages {
    stage('Checkout'){
      steps {
         git branch: 'main', url: 'https://github.com/meeraparigi/Project-CICD.git'  
      }
    }

    stage('Initialize Terraform'){
      steps {
         sh 'terraform init'
      }
    }

    stage('Validate Terraform Code'){
      steps {
         sh 'terraform validate'
      }
    }

    stage('Create Terraform Plan'){
      steps {
         sh 'terraform plan'
      }
    }

    stage('Create Inforastructure with Terraform'){
      steps {
         sh 'terraform apply -auto-approve'
      }
    }
  }
}
