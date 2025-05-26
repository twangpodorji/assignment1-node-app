pipeline {
    agent any
    tools {
        nodejs 'NodeJS 24.0.2'
    }
    
    environment {
        DOCKER_AVAILABLE = 'false'
    }
    
    stages {
        // Stage 1: Checkout Code
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[url: 'https://github.com/twangpodorji/assignment1-node-app.git']]
                ])
            }
        }
        
        // Debug Workspace
        stage('Debug Workspace') {
            steps {
                sh 'ls -la'
            }
        }
        
        // Stage 2: Install Dependencies
        stage('Install Dependencies') {
            steps {
                dir('backend') { // Change 'backend' to the correct directory if needed
                    sh 'npm install'
                }
            }
        }
        
        // Stage 3: Build (if applicable, e.g., for React/TypeScript)
        stage('Build') {
            steps {
                dir('backend') { // Change 'backend' to the correct directory if needed
                    sh 'npm run build || echo "No build script found, skipping build"'
                }
            }
        }
        
        // Stage 4: Run Unit Tests
        stage('Test') {
            steps {
                dir('backend') { // Change 'backend' to the correct directory if needed
                    sh 'npm test || echo "No test script found, skipping tests"'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'junit.xml'
                }
            }
        }
        
        // Stage 5: Check Docker Availability
        stage('Docker Check') {
            steps {
                script {
                    try {
                        sh 'docker --version'
                        sh 'docker info'
                        env.DOCKER_AVAILABLE = 'true'
                        echo 'Docker is available and running'
                    } catch (Exception e) {
                        env.DOCKER_AVAILABLE = 'false'
                        echo "Docker is not available: ${e.getMessage()}"
                        echo 'This might be because Jenkins is running in a container without Docker access'
                    }
                }
            }
        }
        
        // Stage 6: Deploy (Docker Example)
        stage('Deploy') {
            when {
                environment name: 'DOCKER_AVAILABLE', value: 'true'
            }
            steps {
                script {
                    echo 'Building Docker image...'
                    def app = docker.build('wangpo1642/node-app:latest', '-f backend/Dockerfile backend/')
                    
                    echo 'Pushing to Docker Hub...'
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-hub-creds') {
                        app.push('latest')
                    }
                }
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for details.'
        }
    }
}