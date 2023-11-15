def imageName="192.168.44.44:8082/docker_repo/frontend"
def dockerTag=""
def deckerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"

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
        stage("stage SQ") {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
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
