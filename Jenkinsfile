pipeline {
    agent any

    stages {
        stage ('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("fabricioveronez/page-labs:${env.BUILD_ID}", '-f ./src/Dockerfile ./src') 
                }                
            }
        }

        stage ('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Autenticando no Azure') {
            environment {
                AZURE_SERVICE_PRINCIPAL_NAME = credentials("AZURE_SERVICE_PRINCIPAL_NAME")
                AZURE_SECRET = credentials("AZURE_SECRET")
                AZURE_TENANT = credentials("AZURE_TENANT")
            }
            steps {
                sh("az login --service-principal -u $AZURE_SERVICE_PRINCIPAL_NAME -p $AZURE_SECRET --tenant $AZURE_TENANT")
            }
        }
        
        stage('Deploy Azure Container Apps') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                sh("az containerapp update --name live-pipe --resource-group live-pipe --image fabricioveronez/page-labs:${env.BUILD_ID}")
            }
        }
    }
}