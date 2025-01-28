pipeline {
    agent any

    environment {
        DOCKER_USERNAME = credentials('docker-username')  // ID ваших Jenkins credentials
        DOCKER_PASSWORD = credentials('docker-password')
        SSH_PRIVATE_KEY = credentials('ssh-private-key')
        SSH_PUBLIC_KEY = credentials('ssh-public-key')
        SERVER_USER = credentials('server-user')
        SERVER_HOST = credentials('server-host')
        EMAIL_USER = credentials('email-user')
        EMAIL_PASSWORD = credentials('email-password')
        EMAIL_RECEIVER = credentials('email-receiver')
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code..."
                checkout scm
            }
        }

        stage('Test .NET Code') {
            steps {
                script {
                    try {
                        sh '''
                            dotnet restore ./BlazorAppFront/BlazorAppFront.csproj
                            dotnet test --no-build --verbosity normal ./BlazorAppFront/BlazorAppFront.csproj
                        '''
                    } catch (Exception e) {
                        writeFile file: 'test-error.log', text: "Test failed"
                        error("Tests failed")
                    }
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker-credentials-id']) {
                        sh '''
                            docker build -t ${DOCKER_USERNAME}/myapp:${BUILD_ID} ./BlazorAppFront/
                            docker push ${DOCKER_USERNAME}/myapp:${BUILD_ID}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    // Подготовка ключей SSH
                    sh '''
                        mkdir -p ~/.ssh
                        echo "${SSH_PRIVATE_KEY}" > ~/.ssh/id_rsa
                        echo "${SSH_PUBLIC_KEY}" > ~/.ssh/id_rsa.pub
                        chmod 600 ~/.ssh/id_rsa
                        ssh-copy-id -i ~/.ssh/id_rsa.pub ${SERVER_USER}@${SERVER_HOST}
                    '''

                    // Выполнение команд через SSH
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${SERVER_USER}@${SERVER_HOST} << EOF
                        docker pull ${DOCKER_USERNAME}/myapp:${BUILD_ID}
                        docker stop myapp || true
                        docker rm myapp || true
                        docker run -d --name myapp -p 8088:80 ${DOCKER_USERNAME}/myapp:${BUILD_ID}
                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: "${EMAIL_RECEIVER}",
                 subject: "CI/CD Pipeline Completed Successfully",
                 body: """
                 Pipeline completed successfully.
                 All jobs succeeded:
                 - Test .NET Code
                 - Build and Push Docker Image
                 - Deploy to Server
                 """
        }

        failure {
            mail to: "${EMAIL_RECEIVER}",
                 subject: "CI/CD Pipeline Failed",
                 body: """
                 Pipeline failed. Check the logs for more details.
                 """
        }
    }
}
