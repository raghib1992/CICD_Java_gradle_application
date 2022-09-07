pipeline {
    agent any
    tools {
        gradle 'gradle_7'
        jdk 'java-11'
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
    }
}