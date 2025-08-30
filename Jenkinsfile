pipeline {
  agent any
  environment {
    BRANCH_NAME = 'main'
    REPO_URL = 'https://github.com/Prabhupadige03/automation.git'
    TARGET_HOST = 'ubuntu@51.20.190.15'
    TARGET_DIR = '/var/www/html/my-react-app'
  }
  stages {
    stage('Clean Workspace') {
      steps { deleteDir() }
    }
    stage('Clone Repository') {
      steps { git branch: "${BRANCH_NAME}", url: "${REPO_URL}" }
    }
    stage('Install Dependencies') {
      steps {
        sh '''
          rm -rf node_modules package-lock.json
          npm install
        '''
      }
    }
    stage('Build React App') {
      steps { sh 'npm run build' }
    }
    stage('Deploy to Target EC2') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'my-ssh-key-id', keyFileVariable: 'SSH_KEY')]) {
          sh """
            ssh -i $SSH_KEY -o StrictHostKeyChecking=no ${TARGET_HOST} 'sudo rm -rf ${TARGET_DIR} && sudo mkdir -p ${TARGET_DIR} && sudo chown ubuntu:ubuntu ${TARGET_DIR}'
            scp -i $SSH_KEY -o StrictHostKeyChecking=no -r build/* ${TARGET_HOST}:${TARGET_DIR}/
          """
        }
      }
    }
    stage('Restart Server (Optional)') {
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'my-ssh-key-id', keyFileVariable: 'SSH_KEY')]) {
          sh "ssh -i \$SSH_KEY -o StrictHostKeyChecking=no ${TARGET_HOST} 'sudo systemctl restart nginx'"
        }
      }
    }
  }
  post {
    failure { echo 'Pipeline failed!' }
    success { echo 'Pipeline completed successfully!' }
  }
}

