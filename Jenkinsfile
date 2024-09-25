pipeline {
    agent any
    tools {
        nodejs 'node' // 'nodejs' should match the name you configured in Global Tool Configuration
    }
    parameters {
        string(name: 'AppPort', defaultValue: '3000', description: 'Port to run the application')
        string(name: 'RepositoryName', defaultValue: 'bank', description: 'Repository name for Docker image') // Parameterize repository name
        string(name: 'ImageName', defaultValue: 'bank', description: 'Docker image name') // Parameterize image name
    }
    environment {
        DOCKER_IMAGE_NAME = "${params.JFrogURL}/${params.RepositoryName}/${params.ImageName}:${env.BUILD_NUMBER}" // Parameterized repository and image names
    }
    stages {
        stage('Clone') {
            steps {
                git branch: 'test', url: "https://github.com/meki63/DevOps-CI-Project.git"
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs ."
            }
        }
        stage('Backend') {
            steps {
                sh '''
                    mkdir -p $WORKSPACE/app/backend
                    cd $WORKSPACE/app/backend
                    npm install
                '''
            }
        }
        stage('Frontend') {
            steps {
                sh '''
                    mkdir -p $WORKSPACE/app/frontend
                    cd $WORKSPACE/app/frontend
                    npm install
                '''
            }
        }
        stage('Unit Tests - Backend') {
    steps {
        script {
            dir("$WORKSPACE/app/backend") { // Adjust this path if necessary
                sh 'npm install' // Install dependencies
                sh 'npm test' // Run Mocha tests
            }
        }
    }
}
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh '''
                        cd $WORKSPACE/app
                        docker compose build
                    '''
                }
            }
        }
        stage('Deploy to Container') {
            steps {
                script {
                    // Deploy the application using docker-compose
                    sh '''
                        cd $WORKSPACE/app
                        docker compose up -d
                    '''
                }
            }
        }
   post {
        always {
             //Clean up Docker containers after the pipeline finishes
          script {
               sh '''
                    cd $WORKSPACE/app
                    docker compose down
              '''
           }
        }
    }
}
}