pipeline {
    agent any
    tools {
	    maven "maven"

	}
    stages{
        stage('Fetch code') {
          steps{
              git branch: 'master', url: 'https://github.com/Arlette-Yolande/jenkins-webhook-demo.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.jar'
                }
            }
        }
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }

        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonar4.7'
            }
            steps {
               withSonarQubeEnv('sonar') {
                   sh '''${scannerHome}/bin/sonar-scanner -X -Dsonar.projectKey=deployapp \
                   -Dsonar.projectName=deployapp \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/* '''
              }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: '13.91.222.198:8081',
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: 'deployapp',
                  credentialsId: 'NexusLogin',
                  artifacts: [
                    [artifactId: 'deployapp',
                     classifier: '',
                     file: 'target/positionreceiver-0.0.1-SNAPSHOT.jar',
                     type: 'jar']
    ]
 )
            }
        }



    }


    }
