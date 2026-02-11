pipeline {
    agent any
    
    environment {
        AZURE_VM_IP = '20.82.160.97'
        AZURE_VM_USER = 'msa'
        DEPLOY_PATH = '/var/www/devops-dotnet-demo'
        SERVICE_NAME = 'devops-dotnet-demo'
        DEPLOY_ZIP = 'deploy.zip'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                bat 'dotnet restore DevOpsDotNetDemo.sln'
                bat 'dotnet build DevOpsDotNetDemo.sln --configuration Release'
                bat 'dotnet publish DevOpsDotNetDemo.csproj -c Release -o ./publish'
                
                // Create deploy.zip from publish folder
                bat 'powershell -Command "Compress-Archive -Path ./publish/* -DestinationPath ./%DEPLOY_ZIP% -Force"'
            }
        }
        
        stage('Test SSH Connection') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'azure-vm-ubuntu-deplloyment-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    bat "ssh -i %SSH_KEY% -o StrictHostKeyChecking=no %AZURE_VM_USER%@%AZURE_VM_IP% echo Connected!"
                }
            }
        }
        
        stage('Deploy to Azure VM') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'azure-vm-ubuntu-deplloyment-ssh-key', keyFileVariable: 'SSH_KEY')]) {
                    script {
                        // Optional: fix key permissions if Jenkins user needs it
                        bat "icacls %SSH_KEY% /inheritance:r"
                        bat "icacls %SSH_KEY% /grant:r \"%USERNAME%:R\""
                        
                        // Copy deploy.zip to Azure VM
                        bat "scp -i %SSH_KEY% -o StrictHostKeyChecking=no \"%WORKSPACE%\\%DEPLOY_ZIP%\" %AZURE_VM_USER%@%AZURE_VM_IP%:/tmp/"
                        
                        // Deploy: stop service, clear folder, unzip, set permissions, start service
                        bat """
                            ssh -i %SSH_KEY% -o StrictHostKeyChecking=no %AZURE_VM_USER%@%AZURE_VM_IP% "sudo systemctl stop %SERVICE_NAME% &&
                            sudo rm -rf %DEPLOY_PATH%/* &&
                            sudo unzip -o /tmp/%DEPLOY_ZIP% -d %DEPLOY_PATH% &&
                            sudo chown -R www-data:www-data %DEPLOY_PATH% &&
                            sudo systemctl start %SERVICE_NAME%"
                        """
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo "Deployment completed! Access your app at http://${AZURE_VM_IP}"
        }
        failure {
            echo "Deployment failed. Check the logs for details."
        }
    }
}
