 def COLOR_MAP = [
	'SUCCESS': 'good',
	'FAILURE': 'danger',
 ]

 
 pipeline {
    agent any

    stages{
    
        stage('Fetch code') {
          steps{
              git branch: 'vp-rem', url:'https://github.com/ismail-cs/CI-CD_Project.git'
          }  
        }

        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Now Archiving."
                    archiveArtifacts artifacts: '**/*.war'
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
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile_test \
                   -Dsonar.projectName=vprofile_test \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
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
        
        stage("Upload Artifact") {
        	steps {
        		
        		nexusArtifactUploader(
				nexusVersion: 'nexus3',
				protocol: 'http',
				nexusUrl: '172.31.52.210:8081',
				groupId: 'QA',
				version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
				repository: 'vprofile_ci_2',
				credentialsId: 'nexuslogin',
					artifacts: [
						[artifactId: 'vproapp',
						 classifier: '',
						 file: 'target/vprofile-v2.war',
						 type: 'war']
					]
			 	)	
        	}
        }



    }
    
    post {
		always {
			echo 'Slack Notification.'
			slackSend channel: 'team_a',
				color: COLOR_MAP[currentBuild.currentResult],
				message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
		}
	}
}


