pipeline {
    agent any

    environment {
        AZURE_CREDENTIALS_ID = 'azure-service-principal'
        RESOURCE_GROUP = 'rg-jenkins'
        APP_SERVICE_NAME = 'webapijenkinsaditya108108'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/jnaditya108/react-app-jenkins.git'
            }
        }

        stage('Azure Login & Terraform Init') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: AZURE_CREDENTIALS_ID,
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID'
                )]) {
                    bat """
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        set ARM_CLIENT_ID=%AZURE_CLIENT_ID%
                        set ARM_CLIENT_SECRET=%AZURE_CLIENT_SECRET%
                        set ARM_SUBSCRIPTION_ID=%AZURE_SUBSCRIPTION_ID%
                        set ARM_TENANT_ID=%AZURE_TENANT_ID%
                        cd terraform
                        terraform init
                    """
                }
            }
        }

        stage('Terraform Plan & Apply') {
            steps {
                bat """
                    cd terraform
                    terraform plan -out=tfplan
                    terraform apply -auto-approve tfplan
                """
            }
        }

        stage('Build React App') {
            steps {
                dir('my-react-app') {
                    bat 'npm install'
                    bat 'npm run build'
                }
            }
        }

        stage('Deploy to Azure') {
            steps {
                withCredentials([azureServicePrincipal(
                    credentialsId: AZURE_CREDENTIALS_ID,
                    subscriptionIdVariable: 'AZURE_SUBSCRIPTION_ID',
                    clientIdVariable: 'AZURE_CLIENT_ID',
                    clientSecretVariable: 'AZURE_CLIENT_SECRET',
                    tenantIdVariable: 'AZURE_TENANT_ID'
                )]) {
                    bat """
                        az login --service-principal -u %AZURE_CLIENT_ID% -p %AZURE_CLIENT_SECRET% --tenant %AZURE_TENANT_ID%
                        powershell -Command Compress-Archive -Path ./my-react-app/build/* -DestinationPath ./my-react-app/build.zip -Force
                        az webapp deploy --resource-group %RESOURCE_GROUP% --name %APP_SERVICE_NAME% --src-path ./my-react-app/build.zip --type zip
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment Successful!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}
