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
                        sh 'dotnet publish --configuration Release --runtime win-x64 --self-contained false'
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
                // Deploy to the remote IIS server for branch1
                withCredentials([sshUserPrivateKey(credentialsId: 'e42c2480-89f9-4e15-a966-21bcc8ab478e', keyFileVariable: 'SSH_KEY')]) {
                    try {
                        // Stop IIS application pool or entire IIS if necessary
                        sh '''
                            ssh -i $SSH_KEY admin@192.168.5.25 "powershell -Command 'Stop-WebAppPool -Name news'"
                        '''
                        
                        // Copy files to the remote server (use rsync for reliability)
                        sh '''
                            rsync -avz --delete -e "ssh -i $SSH_KEY" /var/lib/jenkins/workspace/multiBranch_pipeline_new_branch1/bin/Release/net8.0/win-x64/ admin@192.168.5.25:/C:/win-x64
                        '''
                        
                        // Restart IIS application pool after the files are copied
                        sh '''
                            ssh -i $SSH_KEY admin@192.168.5.25 "powershell -Command 'Start-WebAppPool -Name news'"
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
