pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code..."
                git url: 'git@github.com:Armen8793/gitops-dotnet-helm-argocd.git', 
                    credentialsId: 'github-ssh-key', 
                    branch: 'main' 
            }
        }

        stage('Install .NET SDK') {
            steps {
                script {
                    echo "Installing .NET SDK..."
                    sh '''
                        wget https://download.visualstudio.microsoft.com/download/pr/3d799a68-d9fd-40f9-bd0d-82e24571eb7a/70d217a3495e1f0286a4f49c77f2e9fc/dotnet-sdk-6.0.400-linux-x64.tar.gz
                        mkdir -p $HOME/dotnet
                        tar -zxf dotnet-sdk-6.0.400-linux-x64.tar.gz -C $HOME/dotnet
                        echo "export PATH=\$PATH:$HOME/dotnet" >> ~/.bashrc
                        source ~/.bashrc
                    '''
                }
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
