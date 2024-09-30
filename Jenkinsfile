pipeline {
    agent any

    environment {
        DOTNET_ROOT = '/usr/bin/dotnet'
        ASPNETCORE_ENVIRONMENT = 'Production'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the branch that triggered the build
                git branch: env.BRANCH_NAME, url: 'https://github.com/Nehyan9895/dotnet-CI-CD.git'
            }
        }

        stage('Restore Dependencies') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --configuration Release'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test'
            }
        }

        stage('Publish') {
            steps {
                script {
                    // Publish based on the branch name
                    if (env.BRANCH_NAME == 'branch1') {
                        sh 'dotnet publish --configuration Release -o ./publish/client1'
                    } else if (env.BRANCH_NAME == 'branch2') {
                        sh 'dotnet publish --configuration Release -o ./publish/client2'
                    } else if (env.BRANCH_NAME == 'branch3') {
                        sh 'dotnet publish --configuration Release -o ./publish/client3'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'branch1') {
                        // Use username and password for SSH
                        withCredentials([usernamePassword(credentialsId: 'remote-server-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            try {
                                // Copy files to the remote server using sshpass
                                sh '''
                                    sshpass -p "$PASSWORD" scp -o StrictHostKeyChecking=no -r ./publish/client1/* $USERNAME@192.168.5.25:C:/CICDTest1
                                '''
                                // Restart the application pool
                                sh '''
                                    sshpass -p "$PASSWORD" ssh -o StrictHostKeyChecking=no $USERNAME@192.168.5.25 "powershell -Command 'Restart-WebAppPool -Name \"CICDTest1\"'"
                                '''
                            } catch (Exception e) {
                                // Handle errors
                                echo "Deployment failed: ${e.message}"
                                currentBuild.result = 'FAILURE'
                            }
                        }
                    } else if (env.BRANCH_NAME == 'branch2') {
                        // Local Nginx server deployment for branch2
                        sh 'sudo systemctl restart client2-app'
                    } else if (env.BRANCH_NAME == 'branch3') {
                        // Local Nginx server deployment for branch3
                        sh 'sudo systemctl restart client3-app'
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/publish/**'
        }

        failure {
            mail to: 'your-email@example.com',
                 subject: "Build failed: ${currentBuild.fullDisplayName}",
                 body: "Something went wrong: ${env.BUILD_URL}"
        }
    }
}
