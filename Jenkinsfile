pipeline {
  agent any

  environment {
      AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
      AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
      PEM_FILE = credentials('ansible-ssh-key')
      INVENTORY_FILE = 'inventory.ini'
      PLAYBOOK_FILE = 'install_docker.yaml'
  }

  stages {
    stage('Checkout'){
      steps {
          checkout scm
      }
    }

    stage('Run Ansible Playbook'){
      steps{
          sh """
            chmod 600 $PEM_FILE
            ansible -i ${INVENTORY_FILE} -m all ping
            ansible-playbook -i ${INVENTORY_FILE} ${PLAYBOOK_FILE} --private-key $PEM_FILE
          """
      }
    }

    stage('Terraform Init') {
      steps {
        dir('terraform') {
          sh 'terraform init'
        }
      }
    }

    stage('Validate Terraform Code') {
      steps {
        dir('terraform') {
          sh 'terraform validate'
        }
      }
    }
   
    stage('Create Terraform Plan') {
      steps {
        dir('terraform') {
          sh 'terraform plan'
        }
      }
    }

    stage('Create Infrastructure with Terraform') {
      steps {
        dir('terraform') {
          sh 'terraform apply -auto-approve'
        }
      }
    }
  }
}
