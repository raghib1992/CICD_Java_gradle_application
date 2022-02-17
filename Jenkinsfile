pipeline {
    agent {
        node {
            label 'slaveTwo'
        }
    }

    tools {
        gradle 'gradle-7-4'
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