pipeline {
	agent any
	tools {
		maven 'maven-latest'
		jdk 'java-1.8.0'
	}
	environment {
		PATH = """${sh(
				returnStdout: true,
				script: 'chmod +x $WORKSPACE/.build/*; printf $WORKSPACE/.build:$PATH'
			)}"""

		MAVEN_SETTINGS = """${sh(
				returnStdout: true,
				script: 'printf $WORKSPACE/.mvn/settings.xml'
			)}"""
		REPOS = """${sh(
				returnStdout: true,
				script: 'REPOS=repo-releases; if [ $GIT_BRANCH != master ]; then REPOS=$REPOS,repo-development; fi; printf $REPOS'
			)}"""

		CHANGES_MAVEN_GENERIC = """${sh(
				returnStdout: true,
				script: '.build/git-check-for-change maven-generic/ mvn-generic'
			)}"""
		CHANGES_MAVEN_JAVA = """${sh(
				returnStdout: true,
				script: '.build/git-check-for-change maven-java/ mvn-java'
			)}"""
	}
	stages {
		stage('Initialize') {
			steps {
				sh 'echo "PATH = ${PATH}"'
				sh 'echo "M2_HOME = ${M2_HOME}"'
				sh 'printenv | sort'
				sh '''
					echo CHANGES_MAVEN_GENERIC=$(git-check-for-change maven-generic/ mvn-generic)
					echo CHANGES_MAVEN_JAVA=$(git-check-for-change maven-java/ mvn-java)
				'''
			}
		}
		stage('Update Maven Repo') {
			when {
				anyOf {
					environment name: 'CHANGES_MAVEN_GENERIC', value: '1'
					environment name: 'CHANGES_MAVEN_JAVA', value: '1'
				}
			}
			steps {
				dir(path: 'maven-generic') {
					sh 'mvn-dev -P ${REPOS} dependency:resolve --non-recursive'
				}
			}
		}

		stage('Install Helper - Generic') {
			parallel {
				stage('Maven') {
					when {
						environment name: 'CHANGES_MAVEN_GENERIC', value: '1'
					}
					steps {
						dir(path: 'maven-generic') {
							sh 'mvn-dev -P ${REPOS},install --non-recursive'
							sh 'ls -l target'
						}
					}
				}
			}
		}
		stage('Install Helper - Components') {
			parallel {
				stage('Maven - Java') {
					when {
						environment name: 'CHANGES_MAVEN_JAVA', value: '1'
					}
					steps {
						dir(path: 'maven-java') {
							sh 'mvn-dev -P ${REPOS},install --non-recursive'
							sh 'ls -l target'
						}
					}
				}
			}
		}



		stage('Deploy') {
			parallel {
				stage('Develop') {
					stages {
						stage('maven-generic') {
							when {
								environment name: 'CHANGES_MAVEN_GENERIC', value: '1'
							}
							steps {
								dir(path: 'maven-generic') {
									sh 'mvn-dev -P ${REPOS},dist-repo-development,deploy --non-recursive'
								}
							}
						}
						stage('maven-java') {
							when {
								environment name: 'CHANGES_MAVEN_JAVA', value: '1'
							}
							steps {
								dir(path: 'maven-java') {
									sh 'mvn-dev -P ${REPOS},dist-repo-development,deploy --non-recursive'
								}
							}
						}
					}
				}
				stage('Release') {
					when {
						branch 'master'
					}
					stages {
						stage('maven-generic') {
							when {
								environment name: 'CHANGES_MAVEN_GENERIC', value: '1'
							}
							steps {
								dir(path: 'maven-generic') {
									sh 'mvn-dev -P ${REPOS},dist-repo-releases,deploy-pom-signed --non-recursive'
								}
							}
						}
						stage('maven-java') {
							when {
								environment name: 'CHANGES_MAVEN_JAVA', value: '1'
							}
							steps {
								dir(path: 'maven-java') {
									sh 'mvn-dev -P ${REPOS},dist-repo-releases,deploy-pom-signed --non-recursive'
								}
							}
						}
					}
				}
			}
			post {
				always {
					archiveArtifacts artifacts: '*/target/*.pom', fingerprint: true
					archiveArtifacts artifacts: '*/target/*.asc', fingerprint: true
				}
			}
		}
		
		stage('Stage at Maven-Central') {
			when {
				branch 'master'
			}
			stages {
				// never add : -P ${REPOS} => this is ment to fail here
				stage('maven-generic') {
					when {
						environment name: 'CHANGES_MAVEN_GENERIC', value: '1'
					}
					steps {
						dir(path: 'maven-generic') {
							sh 'mvn-dev -P dist-repo-maven-central,deploy-pom-signed --non-recursive'
						}
					}
				}
				stage('maven-java') {
					when {
						environment name: 'CHANGES_MAVEN_JAVA', value: '1'
					}
					steps {
						dir(path: 'maven-java') {
							sh 'mvn-dev -P dist-repo-maven-central,deploy-pom-signed --non-recursive'
						}
					}
				}
			}
		}
	}
	post {
		cleanup {
			cleanWs()
		}
	}
}
