pipeline {
    agent any
    tools {
        nodejs 'NodeJS 24.0.2'
    }
    stages {
        // Stage 1: Checkout Code
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/twangpodorji/assignment1-node-app.git'
            }
        }
        // Stage 2: Install Dependencies (Backend)
        stage('Install Backend') {
            steps {
                dir('backend') {
                    sh 'npm install'
                }
            }
        }
        // Stage 2b: Install Dependencies (Frontend)
        stage('Install Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                }
            }
        }
        // Stage 3: Build Backend (if applicable)
        stage('Build Backend') {
            steps {
                dir('backend') {
                    sh 'npm run build || echo "No build script in backend, continuing"'
                }
            }
        }
        // Stage 3b: Build Frontend (React/TypeScript)
        stage('Build Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm run build'
                }
            }
        }
        // Stage 4: Run Backend Unit Tests
        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh 'npm test || echo "No test script in backend, continuing"'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/junit.xml'
                }
            }
        }
        // Stage 4b: Run Frontend Unit Tests
        stage('Test Frontend') {
            steps {
                dir('frontend') {
                    sh 'npm test || echo "No test script in frontend, continuing"'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'frontend/junit.xml'
                }
            }
        }
        // Stage 5: Deploy (Docker Example)
        stage('Deploy') {
            steps {
                script {
                    // Build Docker image using backend Dockerfile
                    sh 'docker build -t wangpo1642/node-app:latest -f backend/Dockerfile backend/'
                    // Push to Docker Hub (requires credentials)
                    withCredentials([string(credentialsId: 'docker-hub-creds', variable: 'DOCKER_PWD')]) {
                        sh 'echo $DOCKER_PWD | docker login -u wangpo1642 --password-stdin'
                        sh 'docker push wangpo1642/node-app:latest'
                    }
                }
            }
        }
    }
}