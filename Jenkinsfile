pipeline {
	agent any
	tools {
		maven 'maven-latest'
	}
	stages {
		stage('Install Helper') {
			parallel {
			    stage('Maven') {
			        steps {
			        	dir(path: 'maven') {
			        		sh 'mvn -P helper-install --non-recursive'
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
			        		sh 'mvn -P helper-install --non-recursive'
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
			        		sh 'mvn -P dist-repo-development,helper-deploy'
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
