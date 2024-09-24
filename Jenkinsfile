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
                    // Restart the service based on the branch name
                    if (env.BRANCH_NAME == 'branch1') {
                        sh 'sudo systemctl restart client1-app'
                    } else if (env.BRANCH_NAME == 'branch2') {
                        sh 'sudo systemctl restart client2-app'
                    } else if (env.BRANCH_NAME == 'branch3') {
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
