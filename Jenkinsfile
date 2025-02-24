pipeline {
    agent any

    tools {
        nodejs "nodejs"
    }

    environment {
        DEPLOY_SSH_KEY = credentials('AWS_INSTANCE_SSH')  // SSH Key for Deployment
        PRODUCTION_IP_ADDRESS = '52.204.238.172'  // Replace with actual IP or DNS
    }

    stages {
        stage('Install Packages') {
            steps {
                sh 'yarn install'
            }
        }

        stage('Run the App') {
            steps {
                sh 'npx pm2 start --interpreter babel-node src/app.js --name todos-app'
                sleep 5
            }
        }

        stage('Test the App') {
            steps {
                sh 'curl --fail http://localhost:3000/health || exit 1'
            }
        }

        stage('Stop the App') {
            steps {
                sh 'npx pm2 stop todos-app || echo "App not running"'
            }
        }

        stage('Add Host to known_hosts') {
            steps {
                sh '''
                    mkdir -p /var/lib/jenkins/.ssh
                    chmod 700 /var/lib/jenkins/.ssh
                    ssh-keyscan -H $PRODUCTION_IP_ADDRESS >> /var/lib/jenkins/.ssh/known_hosts
                    chmod 600 /var/lib/jenkins/.ssh/known_hosts
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    ssh -v -i $DEPLOY_SSH_KEY ubuntu@$PRODUCTION_IP_ADDRESS '
                        if [ ! -d "todos-app" ]; then
                            git clone https://github.com/AhmadMazaal/todos-app.git todos-app
                            cd todos-app
                        else
                            cd todos-app
                            git pull
                        fi

                        yarn install

                        if npx pm2 describe todos-app > /dev/null ; then
                            npx pm2 restart todos-app
                        else
                            npx pm2 start --interpreter babel-node src/app.js --name todos-app
                        fi
                    '
                '''
            }
        }
    }

    post {
        always {
            sh 'npx pm2 delete todos-app || echo "No PM2 process to delete"'
        }
        failure {
            echo 'The pipeline has failed. Please check the logs.'
        }
        success {
            echo 'Deployment successful! 🚀'
        }
    }
}
