pipeline {
    agent any
    parameters {
        string(name: 'SONAR_HOST_URL', defaultValue: 'http://192.168.201.13:9000', description: 'SonarQube host URL')
        string(name: 'SONAR_API_KEY_ID', defaultValue: 'sonar_api_key', description: 'SonarQube API key credentials ID')
    }
    tools {
        nodejs 'node16' // Ensure Node.js 16 is installed on Jenkins
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner' // Assuming SonarQube scanner is configured
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: params.SONAR_API_KEY_ID, variable: 'SONAR_API_KEY')]) {
                    withSonarQubeEnv('sonar') { // 'sonar' should match your SonarQube server configuration name in Jenkins
                        sh """
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectKey=Bank \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=${SONAR_HOST_URL} \
                        -Dsonar.login=$SONAR_API_KEY
                        """
                    }
                }
            }
        }
        
        stage('Start Containers') {
            steps {
                sh 'npm run compose:up'
            }
        }
        stage('Run Integration Tests') {
            steps {
                sh 'npm run test:integration'
            }
        }
        stage('Run E2E Tests') {
            steps {
                sh 'npm run test:e2e'
            }
        }
        // Uncomment if you want to bring the containers down after tests
        /*
        stage('Stop Containers') {
            steps {
                sh 'npm run compose:down'
            }
        }
        */
    }
    post {
        always {
            echo 'Cleaning up resources...'
            // Ensure containers are stopped regardless of build status
            sh 'npm run compose:down || true'
            // Clean up the workspace
            deleteDir() // Deletes the workspace directory
        }
    }
}