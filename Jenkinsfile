pipeline {
    agent any
    tools {
        maven 'Maven 3'  // Ensure this matches the Maven installation name on your Jenkins (set under Manage Jenkins > Global Tool Configuration)
    }
    environment {
        SONARQUBE_AUTH_TOKEN = credentials('sonar-token')
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "13.203.16.254:8081"
        NEXUS_REPOSITORY = "Maven-Artifacts-Repo"
	    NEXUS_REPO_ID    = "Maven-Artifacts-Repo"
        NEXUS_CREDENTIAL_ID = "nexus-credentials"
        ARTVERSION = "${env.BUILD_ID}"
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
                script {
                    pom = readMavenPom file: "pom.xml";
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}");
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path;
                    artifactExists = fileExists artifactPath;
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version} ARTVERSION";
                        nexusArtifactUploader(
                            nexusVersion: NEXUS_VERSION,
                            protocol: NEXUS_PROTOCOL,
                            nexusUrl: NEXUS_URL,
                            groupId: pom.groupId,
                            version: ARTVERSION,
                            repository: NEXUS_REPOSITORY,
                            credentialsId: NEXUS_CREDENTIAL_ID,
                            artifacts: [
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: artifactPath,
                                type: pom.packaging],
                                [artifactId: pom.artifactId,
                                classifier: '',
                                file: "pom.xml",
                                type: "pom"]
                            ]
                        );
                    } 
		    else {
                        error "*** File: ${artifactPath}, could not be found";
                    }
                }
            }
        }
    }
}
