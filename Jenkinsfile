pipeline {
    agent any
    
    environment {
        AZURE_VM_IP = '20.82.160.97'
        AZURE_VM_USER = 'msa'
        DEPLOY_PATH = '/var/www/devops-dotnet-demo'
        SERVICE_NAME = 'devops-dotnet-demo'
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
            }
        }
        
        stage('Deploy to Azure VM') {
            steps {
                script {
                    bat 'powershell Compress-Archive -Path ./publish/* -DestinationPath ./deploy.zip -Force'
                    
                    sshagent(['azure-vm-ubuntu-deplloyment-ssh-key']) {
                        bat "scp -o StrictHostKeyChecking=no ./deploy.zip %AZURE_VM_USER%@%AZURE_VM_IP%:/tmp/"
                        
                        bat """
                            ssh -o StrictHostKeyChecking=no %AZURE_VM_USER%@%AZURE_VM_IP% "
                                sudo systemctl stop %SERVICE_NAME% &&
                                sudo rm -rf %DEPLOY_PATH%/* &&
                                sudo unzip -o /tmp/deploy.zip -d %DEPLOY_PATH% &&
                                sudo chown -R www-data:www-data %DEPLOY_PATH% &&
                                sudo systemctl start %SERVICE_NAME%
                            "
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
            echo "Deployed to http://${AZURE_VM_IP}"
        }
    }
}