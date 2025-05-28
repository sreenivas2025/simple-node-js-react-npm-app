pipeline {
    agent any
    stages {
        stage('install  os deps') {
            steps {
                sh 'apt-get update && apt-get install npm -y'
            }
        }
         stage('install node deps') {
            steps {
                sh 'npm install'
            }
        }

        stage('Install Test Reporter') {
            steps {
                sh 'npm install --save-dev jest-junit'
            }
        }
         stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test -- --ci --reporters=default --reporters=jest-junit'
            }
        }

        stage('Publish Test Results') {
            steps {
                junit 'junit.xml'
            }
        }
        stage("Run Code Analysis"){
            environment {
                SCANNER_HOME = tool 'sonar-scan'
            }
            steps {

                withSonarQubeEnv('SonarServer') {
                   sh '''$SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.projectKey=myNPMapp \
                       -Dsonar.projectName=myNPMapp \
                       -Dsonar.sources=. \
                       -Dsonar.analysis.mode=publish \
                       -Dsonar.projectVersion=${BUILD_NUMBER}-${GIT_COMMIT_SHORT}
                    
                    '''
                }
            }
        } 
        stage("QualityGate"){
         steps{
            timeout(time:5,unit:'MINUTES'){
                waitForQualityGate abortPipeline: true
            }
            environment {
        NEXUS_URL = 'http://nexus:8081'
        NEXUS_REPO = 'node-artifacts'
        NEXUS_CREDENTIALS = credentials('nexus-creds') // Jenkins credentials (Username/Password)
          }
        }
      }
       stage('Package Artifact') {
            steps {
                script {
                    // Create zip of the whole project directory or build folder
                    sh 'zip -r my-node-app.zip .'
                }
                // Archive the zip as a Jenkins artifact (optional)
                archiveArtifacts artifacts: 'my-node-app.zip', fingerprint: true
            }
        }
        stage('Upload to Nexus') {
            steps {
                script {
                    // Upload zip to Nexus repository via curl
                    sh """
                        curl -u ${NEXUS_CREDENTIALS_USR}:${NEXUS_CREDENTIALS_PSW} --upload-file my-node-app.zip ${NEXUS_URL}/repository/${NEXUS_REPO}/my-node-app.zip
                    """
                }
            }
        } 
    }
}