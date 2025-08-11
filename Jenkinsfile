pipeline {
  agent any

  environment {
      AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
      AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
      ANSIBLE_HOST_KEY_CHECKING = 'False'
      PLAYBOOK_FILE = 'install_docker.yaml'
      SECRET_NAME = 'docker-credentials'
      REGION = 'us-east-1'
  }

  stages {
    stage('Checkout'){
      steps {
          checkout scm
      }
    }

    stage('Run Ansible Playbook') {
      steps {
            withCredentials([file(credentialsId: 'ansible-ssh-key', variable: 'PEM_FILE')]) {
              sh '''
                set -e  # Fail if any command fails
                chmod 600 \$PEM_FILE
    
                # Get the public IP of the EC2 instance using AWS CLI and a tag or name filter
                INSTANCE_ID=$(aws ec2 describe-instances \
                --filters "Name=tag:Name,Values=myserver" "Name=instance-state-name,Values=running" \
                --query "Reservations[0].Instances[0].InstanceId" --output text)
    
                PUBLIC_IP=$(aws ec2 describe-instances \
                --instance-ids $INSTANCE_ID \
                --query "Reservations[0].Instances[0].PublicIpAddress" \
                --output text)
    
                echo "[all]" > inventory.ini
                echo "target1 ansible_host=\${PUBLIC_IP} ansible_user=ubuntu ansible_ssh_private_key_file=\$PEM_FILE" >> inventory.ini
    
                echo "🔄 Pinging EC2 instance..."
                ansible -i inventory.ini all -m ping
    
                echo "🚀 Running playbook to install Docker..."
                ansible-playbook -i inventory.ini \${PLAYBOOK_FILE} --private-key \$PEM_FILE
              '''
        }
      }
    }

    stage('Docker Login') {
      steps {
        script {
          // Fetch secret from AWS Secrets Manager
          def dockerCreds = sh(
            script: "aws secretsmanager get-secret-value --secret-id ${SECRET_NAME} --region ${REGION} --query SecretString --output text",
            returnStdout: true
          ).trim()

          // Parse the JSON string returned by the Secrets Manager
          def creds = readJSON text: dockerCreds
          env.DOCKER_USERNAME = creds.username
          env.DOCKER_PASSWORD = cred.password

          // Perform secure Docker Login using the password via stdin
          sh """
            echo "${DOCKER_PASSWORD}" | docker login -u "${DOCKER_USERNAME}" --password-stdin
          """
        }  
      }
    }

    stage('Build and Push Docker Image') {
      steps {
        sh '''
          docker build -t "${DOCKER_USERNAME}"/myapp:latest .
          docker push "${DOCKER_USERNAME}"/myapp:latest
        '''
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
