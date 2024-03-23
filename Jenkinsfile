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
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;

                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}";

                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTIFACT_VERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                // Artifact generated such as .jar, .ear and .war files.
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging]
                            ]
                        );

                    } else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
        
    }
}