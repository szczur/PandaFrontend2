def imageName="szczur123/panda-frontend"
def dockerTag=""
def deckerRegistry=""
def registryCredentials="dockerhub"

pipeline {
    stages {
        stage("stage0") {
            steps {
                echo "abc"
            }
        }
        stage("stage - git") {
            steps {
                //git branch: 'main', url: 'https://github.com/szczur/PandaFrontend2.git'
                checkout scm
            }
        }
        stage("stage - Unit test") {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
        //stage("stage SQ") {
        //    steps {
         //       withSonarQubeEnv('SonarQube') {
         //           sh "${scannerHome}/bin/sonar-scanner"
         //       }
         //   }
        //}
        stage("build image") {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}-${env.GIT_COMMIT.take(7)}"
                    applicationImage = docker.build("$imageName:$dockerTag",".")
                }
            }
        }
        stage("push image") {
            steps {
                script {
                    docker.withRegistry("$deckerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
		stage ('Push to Repo') {
			steps {
				dir('ArgoCD') {
					withCredentials([gitUsernamePassword(credentialsId: 'git', gitToolName: 'Default')]) {
						git branch: 'main', url: 'https://github.com/szczur/ArgoCD.git'
						sh """ cd frontend
						git config --global user.email "github-email"
						git config --global user.name "github-name"
						sed -i "s#$imageName.*#$imageName:$dockerTag#g" deployment.yaml
						git commit -am "Set new $dockerTag tag."
						git diff
						git push origin main
						"""
					}                  
				} 
			}
		}
    }
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
    }


    agent {
      label 'agent'
    }
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = "1"
        scannerHome = tool 'SonarQube'
    }
}
