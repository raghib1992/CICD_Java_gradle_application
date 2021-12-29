pipeline {
    agent any
    stages {
        stage('SONAR Quality Check') {
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar_token') {
                        sh '''
                           chmod +x gradlew;
                           ./gradlew sonarqube
                           '''
                    }
                }
            }
        }
    }
}