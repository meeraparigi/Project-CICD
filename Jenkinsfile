pipeline {
  agent any

  environment {
      AWS_ACCESS_KEY_ID = credentials('aws-access-key-id')
      AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
      ANSIBLE_HOST_KEY_CHECKING = 'False'
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
            withCredentials([file(credentialsId: 'ansible-ssh-key', variable: 'PEM_FILE')]) {
              sh '''
                chmod 600 $PEM_FILE
                cat <<EOF > inventory.ini
                [all]
                target1 ansible_host=3.93.183.119 ansible_user=ubuntu ansible_ssh_private_key_file=$PEM_FILE
                EOF
                ansible -i inventory.ini all -m ping
                ansible-playbook -i inventory.ini ${PLAYBOOK_FILE} --private-key $PEM_FILE
              '''
        }
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
