pipeline {
    agent any
    tools {
        maven 'Maven 3'  // Ensure this matches the Maven installation name on your Jenkins (set under Manage Jenkins > Global Tool Configuration)
    }
    environment {
        SONARQUBE_AUTH_TOKEN = credentials('sonar-token')
        NEXUS_URL = 'http://13.203.16.254:8081/repository/Nexus-Repo-1'
        NEXUS_REPO = 'Nexus-Repo-1'
        NEXUS_CREDENTIALS = credentials('nexus-credentials')  // Define this credential in Jenkins
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
         stage('Deploy to Nexus') {
            steps {
                script {
                    // Deploy the artifact to Nexus using the mvn deploy command
                    // The repository URL and credentials are provided dynamically
                    sh """
                        mvn deploy:deploy-file \
                            -Dfile=single-module-project.jar \  # Path to your artifact
                            -DartifactId=single-module-project \        # Your artifact's ID
                            -Dversion=1.0.0 \                 # Your artifact's version
                            -Dpackaging=jar \                 # Packaging type (e.g., jar, war)
                            -DrepositoryId=${NEXUS_REPO} \ # The repository ID (this can be anything)
                            -Durl=${NEXUS_URL} \         # Nexus repository URL
                            -Dusername=${NEXUS_CREDENTIALS_USR} \   # Nexus username from Jenkins credentials
                            -Dpassword=${NEXUS_CREDENTIALS_PSW}
                    """
                }
            }
        }
    }
    }
}
