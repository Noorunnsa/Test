pipeline {
    agent any
    tools {
        maven 'Maven 3'  // Ensure this matches the Maven installation name on your Jenkins (set under Manage Jenkins > Global Tool Configuration)
    }
    environment {
        SONARQUBE_AUTH_TOKEN = credentials('sonar-token')
        NEXUS_URL = 'http://13.203.16.254:8081/repository/Nexus-Repo-1'
        NEXUS_REPO = 'Nexus-Repo-1'
        NEXUS_CREDENTIALS = 'nexus-credentials'  // Define this credential in Jenkins
    }
    stages {
        stage('Code Checkout') {
            steps {
                script {
                    def GIT_CRED = 'gitlab-access-token'
                    def GIT_REPO = 'https://gitlab.com/lg332802/LGDOP.git'
                    checkout([
                        $class: 'GitSCM', 
                        branches: [[name: 'main']], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '/var/jenkins_home/workspace/CI-Pipeline/']], 
                        submoduleCfg: [], 
                        userRemoteConfigs: [[credentialsId: GIT_CRED, url: GIT_REPO]]
                    ])
                    sh 'echo $WORKSPACE'
                }
            }
        }
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv ('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Sonarqube \
  -Dsonar.sources=. \
  -Dsonar.host.url=http://13.203.16.254:9000 \
  -Dsonar.token=${SONARQUBE_AUTH_TOKEN} \
  -Dsonar.exclusions=**/*.java"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 5 , unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build with Maven') {
            steps {
                script {
                    dir('/var/jenkins_home/workspace/CI-Pipeline/single-module') {
                        sh 'mvn clean install'
                    }
               }
            }   
        }
         stage('Upload Artifact to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusUrl: "${NEXUS_URL}",
                    repository: "${NEXUS_REPO}",
                    credentialsId: "${NEXUS_CREDENTIALS}",
                    #groupId: 'com.yourcompany',
                    artifactId: 'single-module-project',
                    version: '1.0.0',
                    file: 'target/single-module-project.jar',
                    type: 'jar'
                )
            }
        }
    }
}
