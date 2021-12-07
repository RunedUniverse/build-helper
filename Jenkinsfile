pipeline {
	agent any
	tools {
		maven 'Maven 3.6.3'
	}
	stages {
		stage('Install Helper') {
			parallel {
			    stage('Maven') {
			        steps {
			        	dir(path: 'maven') {
			        		sh 'mvn -P info,helper-install --non-recursive'
			        	}						
					}
			    }
			}	
		}
		stage('Install Helper - Components') {
			parallel {
			    stage('Maven - Java') {
			        steps {
			        	dir(path: 'maven/build-helper-java') {
			        		sh 'mvn -P info,helper-install --non-recursive'
			        	}						
					}
			    }
			}	
		}
		stage('Deploy Helper') {
			parallel {
			    stage('Maven') {
			        steps {
			        	dir(path: 'maven') {
			        		sh 'mvn -P info,repo-development,helper-deploy'
			        	}						
					}
			    }
			}
		}
	}
	post {
		success {
		    archiveArtifacts artifacts: 'maven/**/pom.xml', fingerprint: true
		}
		cleanup {
			cleanWs()
		}
	}
}
