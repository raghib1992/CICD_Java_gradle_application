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
                        // sh 'chmod +x gradlew'
                        sh 'gradle sonarqube'
                    }

                    timeout(time: 2, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}