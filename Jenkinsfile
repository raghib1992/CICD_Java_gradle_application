pipeline {
    agent any
    tools {
        gradle 'gradle_7'
        jdk 'java-11'
    }

    environment {
        VERSION = "${env.BUILD_ID}"
        REPO_HOST = "3.110.131.39:8085"
    }

    stages {
        stage ('Sonar Code Analysis') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube-server') {
                        sh 'gradle sonarqube'
                    }
                    timeout(3) {
                        def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage ('Docker Build and Docker Push') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'docker-repo', variable: 'repo_credential')]) {
                        sh '''
                            docker build -t ${REPO_HOST}/myapp:${VERSION} .
                            docker login -u admin -p $repo_credential ${REPO_HOST}
                            docker push ${REPO_HOST}/myapp:${VERSION}
                            docker rmi ${REPO_HOST}/myapp:${VERSION}
                        '''
                    }
                }
            }
        }
    }
}