pipeline {
    agent any
    environment {
        DOCKER_HOME = '/usr/bin/docker'  
    }
    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code..."
                git url: 'git@github.com:Armen8793/gitops-dotnet-helm-argocd.git', 
                    credentialsId: 'github-ssh-key', 
                    branch: 'main' 
            }
        }

        stage('Login to Docker') {
            steps {
                sh '/usr/bin/docker login -u $DOCKER_CREDENTIALS_USR -p $DOCKER_CREDENTIALS_PSW'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                sh '''
                    /usr/bin/docker build -t ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID} ./BlazorAppFront/
                    /usr/bin/docker push ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID}
               '''
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${SERVER_CREDENTIALS_USR}@${SERVER_CREDENTIALS_PSW} << EOF
                        /usr/bin/docker pull ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID}
                        /usr/bin/docker stop myapp || true
                        /usr/bin/docker rm myapp || true
                        /usr/bin/docker run -d --name myapp -p 8088:80 ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID}
                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: "petrosyanarmen723@gmail.com",
                 subject: "CI/CD Pipeline Completed Successfully",
                 body: "Pipeline completed successfully. All jobs succeeded."
        }

        failure {
            mail to: "petrosyanarmen723@gmail.com",
                 subject: "CI/CD Pipeline Failed",
                 body: "Pipeline failed. Check the logs for more details."
        }
    }
}
