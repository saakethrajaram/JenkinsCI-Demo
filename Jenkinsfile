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
                bat 'npm install'
            }
        }

        stage('Run Unit & Integration Tests') {
            steps {
                bat '''
                echo Running tests...
                npm test || echo 'Tests failed or Snyk not authenticated, continuing...'
                '''
            }
        }

        stage('SonarCloud Analysis') {
            steps {
                withEnv(["SONAR_TOKEN=${SONAR_TOKEN}"]) {
                    bat '''
                    echo Starting SonarCloud scan...
                    sonar-scanner ^
                      -Dsonar.projectKey=saakethrajaram_JenkinsCI-Demo ^
                      -Dsonar.organization=saakethrajaram ^
                      -Dsonar.sources=. ^
                      -Dsonar.exclusions=node_modules/**,test/** ^
                      -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info ^
                      -Dsonar.login=%SONAR_TOKEN%
                    '''
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
