pipeline {
    agent any

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN') // SonarCloud token
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/saakethrajaram/JenkinsCI-Demo.git',
                    branch: 'main',
                    credentialsId: 'git-creds'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Run Unit & Integration Tests') {
            steps {
                sh 'npm test || true'
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withSonarQubeEnv('SonarCloud') {
                    sh """
                        sonar-scanner \
                          -Dsonar.projectKey=saakethrajaram_JenkinsCI-Demo \
                          -Dsonar.organization=saakethrajaram \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=node_modules/**,test/** \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                          -Dsonar.login=$SONAR_TOKEN
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/target/*.log', allowEmptyArchive: true
        }
    }
}
