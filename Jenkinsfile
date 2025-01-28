pipeline {
    agent any

    environment {
        DOCKER_CREDENTIALS = credentials('docker-credentials')
        SERVER_CREDENTIALS = credentials('server-credentials') 
        GIT_CREDENTIALS = credentials('github-ssh-key')        
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code..."
                git url: 'git@github.com:ваш_пользователь/ваш_репозиторий.git', credentialsId: 'github-ssh-key'
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
                    withDockerRegistry([url: 'https://index.docker.io/v1/', credentialsId: 'docker-credentials']) {
                        sh '''
                            docker build -t ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID} ./BlazorAppFront/
                            docker push ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID}
                        '''
                    }
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                script {
                    sh '''
                        ssh -o StrictHostKeyChecking=no ${SERVER_CREDENTIALS_USR}@${SERVER_CREDENTIALS_PSW} << EOF
                        docker pull ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID}
                        docker stop myapp || true
                        docker rm myapp || true
                        docker run -d --name myapp -p 8088:80 ${DOCKER_CREDENTIALS_USR}/myapp:${BUILD_ID}
                        EOF
                    '''
                }
            }
        }
    }

    post {
        success {
            mail to: "receiver@example.com",
                 subject: "CI/CD Pipeline Completed Successfully",
                 body: "Pipeline completed successfully. All jobs succeeded."
        }

        failure {
            mail to: "receiver@example.com",
                 subject: "CI/CD Pipeline Failed",
                 body: "Pipeline failed. Check the logs for more details."
        }
    }
}
