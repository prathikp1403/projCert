pipeline {
  agent any
  environment {
    TEST_HOST = "<NEW_TEST_IP>"
    TEST_USER = "test"
    SSH_OPTS  = "-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"
  }
  triggers { pollSCM('H/2 * * * *') } // auto build on pushes to master (~every 2 min)

  stages {
    stage('Checkout (master only)') {
      when { branch 'master' }
      steps { checkout scm }
    }

    stage('Job 1: Install puppet agent on TEST') {
      when { branch 'master' }
      steps {
        sh '''
        ssh ${SSH_OPTS} ${TEST_USER}@${TEST_HOST} '
          set -e
          sudo apt-get update -y
          sudo apt-get install -y wget
          if ! command -v puppet >/dev/null 2>&1; then
            wget https://apt.puppet.com/puppet7-release-jammy.deb
            sudo dpkg -i puppet7-release-jammy.deb
            sudo apt-get update -y
            sudo apt-get install -y puppet-agent
          fi
          puppet --version || /opt/puppetlabs/bin/puppet --version || true
        '
        '''
      }
    }

    stage('Job 2: Install Docker on TEST (Ansible)') {
      when { branch 'master' }
      steps {
        sh '''
        cd ansible
        ansible-playbook -i inventory.ini install_docker.yml
        '''
      }
    }

    stage('Job 3: Pull repo on TEST, build & deploy container (Ansible)') {
      when { branch 'master' }
      steps {
        sh '''
        cd ansible
        ansible-playbook -i inventory.ini deploy_app.yml
        '''
      }
    }
  }

  post {
    failure {
      echo 'Job 4: Cleanup on TEST (delete running container)'
      sh 'ssh ${SSH_OPTS} ${TEST_USER}@${TEST_HOST} "sudo docker rm -f wallet || true"'
    }
  }
}

