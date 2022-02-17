pipeline {
    agent {
        node {
            label 'slaveTwo'
        }
    }

    stages {
        stage ('Sonar Quality Test') {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'gradle sonarqube'
                    }
                }
            }
        }
    }
}