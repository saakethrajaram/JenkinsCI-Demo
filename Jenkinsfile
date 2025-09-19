pipeline {
    agent any

    tools {
        sonarQube 'SonarScanner' // must match the name configured in Jenkins
    }

    environment {
        SONAR_TOKEN = credentials('SONAR_TOKEN')   // SonarCloud token
        SNYK_TOKEN  = credentials('SNYK_TOKEN')    // Snyk token (if using)
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/saakethrajaram/JenkinsCI-Demo.git', branch: 'main'
            }
        }

        stage('Install Dependencies') {
            steps {
                // For Node.js project
                sh 'npm install'
            }
        }

        stage('Run Unit & Integration Tests') {
            steps {
                // Continue pipeline even if some tests fail
                sh 'npm test || true'
            }
        }

        stage('Snyk Security Scan') {
            steps {
                sh 'npm install -g snyk'
                sh 'snyk auth $SNYK_TOKEN'
                sh 'snyk test || true'   // captures vulnerabilities without breaking pipeline
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
            // Archive logs for reference
            archiveArtifacts artifacts: '**/target/*.log', allowEmptyArchive: true
        }
    }
}
