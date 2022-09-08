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
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
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
        stage ('Helm chart config analysis by datree'){
            steps{
                script {
                    dir('kubernetes') {
                        withEnv(['DATREE_TOKEN=f892607c-2509-40a6-b42b-c82442379f74']) {
                            sh 'helm datree test myapp/'
                        }
                    }
                }
            }    
        }
        stage ('Helm Chart Push') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'docker-repo', variable: 'repo_credential')]) {
                        dir('kubernetes') {
                            sh '''
                            helmversion=$(helm show chart myapp | grep version | cut -d: -f2 | tr -d ' ')
                            tar -czvf myapp-0.2.0.tgz myapp/                         
                            curl -u admin:$repo_credential http://172.31.37.20:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                        '''
                        }
                        
                    }
                }
            }
        }
    }
    post {
        always {
            // emailext body: 'A Test EMail', recipientProviders: [[$class: 'DevelopersRecipientProvider'], [$class: 'RequesterRecipientProvider']], subject: 'Test'
            mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "azimraghib@gmail.com";
        }
    }
}