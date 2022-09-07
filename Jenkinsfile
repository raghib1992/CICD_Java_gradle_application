pipeline {
    agent any
    tools {
        gradle 'gradle_7'
        jdk 'java-11'
    }

    environment {
        VERSION = ${env.BUILD_ID}
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
                            docker build -t 43.205.228.101:8083/myapp:${VERSION} .
                            docker login -u admin -p $repo_credential 43.205.228.101:8083
                            docker push 43.205.228.101:8083/myapp:${VERSION}
                            docker rmi 43.205.228.101:8083/myapp:${VERSION}
                        '''
                    }
                }
            }
        }
    }
}