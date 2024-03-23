pipeline{
    agent any
    tools {
        maven "M3"
    }
    environment {
    DOCKER_HUB = "challagondlaanilkumar"
    IMAGE_NAME = "bankproject"
    CONTAINER_NAME = "bankproject"
    VERSION = "v${env.BUILD_NUMBER}"
    AWS_ACCOUNT_ID = "339713145834"
    AWS_REGION = "ap-south-1"
    }
    stages{
        stage('Build & Test'){
            steps{
             sh 'mvn clean package'
            }
                
        }
        stage('Code Analysis'){
            steps{
                withSonarQubeEnv(credentialsId: 'sonarbanktoken') {
                    mvn clean verify sonar:sonar -Dsonar.projectKey=bankproject
                }  
            }

        }
    }
}