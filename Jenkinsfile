pipeline {
    agent any

    tools {
        nodejs "nodejs"  // Make sure "nodejs" is properly configured in Jenkins Global Tool Config
    }

    stages {
        stage('Install Packages') {
            steps {
                script {
                    sh 'yarn install'
                }
            }
        }

        stage('Run the App') {
            steps {
                script {
                    sh 'npx pm2 start --interpreter babel-node src/app.js --name todos-app'
                    sleep 5
                }
            }
        }

        stage('Test the App') {
            steps {
                script {
                    sh 'curl --fail http://localhost:3000/health || exit 1'
                }
            }
        }

        stage('Stop the App') {
            steps {
                script {
                    sh 'npx pm2 stop todos-app || echo "App not running"'
                }
            }
        }

        stage('Add Host to known_hosts') {
            steps {
                script {
                    sh '''
                        ssh-keyscan -H $PRODUCTION_IP_ADDRESS >> /var/lib/jenkins/.ssh/known_hosts
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        ssh -v -i $DEPLOY_SSH_KEY ubuntu@$PRODUCTION_IP_ADDRESS '
                            
                            # Clone or update repo
                            if [ ! -d "todos-app" ]; then
                                git clone https://github.com/AhmadMazaal/todos-app.git todos-app
                                cd todos-app
                            else
                                cd todos-app
                                git pull
                            fi

                            # Install dependencies
                            yarn install

                            # Start or restart app with pm2
                            if npx pm2 describe todos-app > /dev/null ; then
                                npx pm2 restart todos-app
                            else
                                npx pm2 start --interpreter babel-node src/app.js --name todos-app
                            fi
                        '
                    """
                }
            }
        }
    }  // <-- Closing stages block

    post {
        always {
            script {
                sh 'npx pm2 delete todos-app || echo "No PM2 process to delete"'
            }
        }
        failure {
            echo 'The pipeline has failed. Please check the logs.'
        }
        success {
            echo 'Deployment successful! 🚀'
        }
    }  // <-- Closing post block
}  // <-- Closing pipeline block
