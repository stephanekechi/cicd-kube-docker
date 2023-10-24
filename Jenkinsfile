pipeline {

	agent any
/*	
	tools {
		maven "MAVEN3"
	}
*/	
	environment {
		registry = "greatness/vprofileapp"
		registryCredential = 'dockerhub'
	}
	
	stages {
	
		stage('BUILD'){
			steps {
				sh 'mvn clean install -DskipTests'
			}
			post {
				success {
					echo 'Now Archiving...'
					archiveArtifacts artifacts: '**/target/*.war'
				}
			}
		}
		
		stage('UNIT TEST'){
			steps {
				sh 'mvn test'
			}
		}
		
		stage('INTEGRATION TEST'){
			steps {
				sh 'mvn checkstyle:checkstyle'
			}
			post {
				success {
					echo 'Generated Analysis Result'
				}
			}
		}
		
		stage('CODE ANALYSIS WITH SONARQUBE'){
			environment {
				scannerHome = tool 'sonar4.7'
			}
			
			steps {
				withSonarQubeEnv('sonar-pro') {
					sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
					-Dsonar.projectName=vprofile-repo \
					-Dsonar.projectVersion=1.0 \
					-Dsonar.sources=src/ \
					-Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
					-Dsonar.junit.reportspath=target/surefire-reports/ \
					-Dsonar.jacoco.reportspath=target/jacoco.exec \
					-Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
				}
				
				timeout(time: 10, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
		
		stage('BUILD APP IMAGE') {
			steps {
				script {
					dockerImage = docker.build registry + ":V$BUILD_NUMBER"
				}
			}
		}
		
		stage ('UPLOAD IMAGE') {
			steps {
				script {
					docker.withRegistry('', registryCredential) {
						dockerImage.push("V$BUILD_NUMBER")
						dockerImage.push('latest')
					}
				}
			}
		}
		
		stage ('REMOVE UNUSED DOCKER IMAGE') {
			steps {
				sh "docker rmi $registry:V$BUILD_NUMBER"
			}
		}
		
		stage('KUBERNETES DEPLOY') {
			agent {label 'KOPS'}
			steps {
				sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:v${BUILD_NUMBER} --namespace prod"
			}
		}
	
	}
	

}