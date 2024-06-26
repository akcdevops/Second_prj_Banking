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
    NEXUS_VERSION = "nexus3"
    NEXUS_PROTOCOL = "https"
    NEXUS_URL = "nexus.akcdevops.online"
    NEXUS_REPOSITORY = "bankproject"
    NEXUS_CREDENTIAL_ID = "nexus"
    ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }
    stages{
        stage('git checkout') {
            steps {
                // Get some code from a GitHub repository
                git branch: 'main', url: 'https://github.com/akcdevops/Second_prj_Banking.git'
            }
            post {
                always{
                  slackSend channel: 'jenkins', 
                  color: 'green', 
                  message:"started  JOB_NAME:${env.JOB_NAME} BUILD_NUMBER:${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)",
                  notifyCommitters: true,  
                  teamDomain: 'dwithitechnologies', 
                  tokenCredentialId: 'slack'
                }
            }
        }
        stage('Build & Test'){
            steps{
             sh 'mvn clean package'
            }
                
        }
        stage('Code Analysis'){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarbanktoken') {
                     sh ' mvn clean package sonar:sonar -Dsonar.projectKey=bankproject'
                    }
                }
                  
            }
        }
        stage('Quality Check'){
            steps{
                timeout(time: 1,unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Push Artifact to Nexus'){
            steps {
                dir('/var/lib/jenkins/workspace/Bankins_project/target'){
                    nexusArtifactUploader artifacts: [[artifactId: 'banking', classifier: '', file: 'banking-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'nexus', groupId: 'com.project.staragile', nexusUrl: '143.110.190.60:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'bankproject', version: '0.0.1-SNAPSHOT'
                }
                        
            }
        }
        stage('Docker Build & Publish'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                    script{ 
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                            sh 'docker build -t ${DOCKER_HUB}/${IMAGE_NAME}:${VERSION} . '
                            sh 'docker build -t ${DOCKER_HUB}/${IMAGE_NAME}:latest .'
                            sh 'docker push ${DOCKER_HUB}/${IMAGE_NAME}:${VERSION} && docker push ${DOCKER_HUB}/${IMAGE_NAME}:latest'
                            sh 'docker rmi ${DOCKER_HUB}/${IMAGE_NAME}:${VERSION} && docker rmi ${DOCKER_HUB}/${IMAGE_NAME}:latest'
                            
                            
                        // } 
                    }
                }
                
            }

        }
        
    }
}