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
            steps {
                // 🧹 Clean up leftover files from previous builds
                deleteDir()
            }
        }

        stage('Clone Repository') {
            steps {
                // ✅ Clone the correct branch
                git branch: "${BRANCH_NAME}", url: "${REPO_URL}"
            }
        }

        stage('Install Dependencies') {
            steps {
                // 🧼 Clean potentially corrupted node_modules + lockfile
                sh '''
                    rm -rf node_modules package-lock.json
                    npm install
                '''
            }
        }

        stage('Build React App') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Deploy to Target EC2') {
            steps {
                sh """
                    ssh -o StrictHostKeyChecking=no ${TARGET_HOST} "sudo rm -rf ${TARGET_DIR} && sudo mkdir -p ${TARGET_DIR} && sudo chown ubuntu:ubuntu ${TARGET_DIR}"
 
                    scp -o StrictHostKeyChecking=no -r build/* ${TARGET_HOST}:${TARGET_DIR}/
                """
            }
        }

        stage('Restart Server (Optional)') {
            steps {
                echo "Restarting nginx on target server"
                sh "ssh -o StrictHostKeyChecking=no ${TARGET_HOST} 'sudo systemctl restart nginx'"
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
