pipeline {
    agent any
    tools {
        maven 'Maven 3'  // Ensure this matches the Maven installation name on your Jenkins (set under Manage Jenkins > Global Tool Configuration)
    }
    environment {
        SONARQUBE_AUTH_TOKEN = credentials('sonar-token')
        NEXUS_VERSION = "nexus3"
        SHARED_VOLUME_PATH = "/var/jenkins_home"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "13.203.16.254:8081"
	    NEXUS_REPO_ID    = "Maven-Artifacts-Repo"
        GROUP_ID = "com.example.maven-samples"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
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
                        extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: '/var/jenkins_home/workspace/ci_cd_stack/']], 
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
                    dir('/var/jenkins_home/workspace/ci_cd_stack/single-module/') {
                        sh 'mvn clean install'
                    }
               }
            } 
        }
       stage("Publish to Nexus Repository Manager") {
            steps {
              nexusArtifactUploader(
                   nexusVersion: "${NEXUS_VERSION}",
                   protocol: "${NEXUS_PROTOCOL}",
                   nexusUrl: "${NEXUS_URL}",
                   groupId: "${GROUP_ID}",
                   repository: "${NEXUS_REPO_ID}",
		           version: "${env.BUILD_ID}",
                   credentialsId: "${NEXUS_CREDENTIAL_ID}",
                   artifacts: [ [artifactId: 'single-module-project', classifier: '', file: "single-module/target/single-module-project.jar",  type: 'jar'] ])  
                }
            }
            stage('Download Artifact from Nexus') {
            steps {
                script {
                    // Construct the URL to download the same version from Nexus
                    def downloadUrl = "${env.NEXUS_URL}/${env.GROUP_ID.replace('.', '/')}/single-module-project/${env.BUILD_ID}/single-module-project.jar"

                    // Download the JAR file using curl
                    sh(script: """
                        curl -u admin:NeoHoney@25 -L -o single-module-project.jar ${downloadUrl}
                    """)
                }
            }
        }
        }
    }
