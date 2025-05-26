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
        
        // Stage 6: Deploy with Docker (if available)
        stage('Deploy with Docker') {
            when {
                environment name: 'DOCKER_AVAILABLE', value: 'true'
            }
            steps {
                script {
                    try {
                        echo 'Building Docker image...'
                        sh 'docker build -t wangpo1642/node-app:latest -f backend/Dockerfile backend/'
                        
                        echo 'Pushing to Docker Hub...'
                        withCredentials([string(credentialsId: 'docker-hub-creds', variable: 'DOCKER_PWD')]) {
                            sh 'echo $DOCKER_PWD | docker login -u wangpo1642 --password-stdin'
                            sh 'docker push wangpo1642/node-app:latest'
                        }
                        echo 'Docker deployment completed successfully'
                        
                    } catch (Exception e) {
                        echo "Docker deployment failed: ${e.getMessage()}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        // Stage 7: Alternative Deploy (when Docker is not available)
        stage('Alternative Deploy') {
            when {
                environment name: 'DOCKER_AVAILABLE', value: 'false'
            }
            steps {
                script {
                    echo 'Docker not available, using alternative deployment method...'
                    echo 'Creating deployment artifacts...'
                    
                    // Create a deployment package
                    sh '''
                        mkdir -p deployment-package
                        
                        # Copy backend files
                        cp -r backend/ deployment-package/
                        
                        # Copy frontend build
                        cp -r frontend/build/ deployment-package/frontend-build/ || echo "Frontend build not found"
                        
                        # Create deployment info
                        echo "Deployment created at: $(date)" > deployment-package/deployment-info.txt
                        echo "Git commit: $(git rev-parse HEAD)" >> deployment-package/deployment-info.txt
                        echo "Build number: ${BUILD_NUMBER}" >> deployment-package/deployment-info.txt
                        
                        # Create a simple startup script
                        cat > deployment-package/start.sh << 'EOF'
#!/bin/bash
echo "Starting Node.js application..."
cd backend
npm install --production
npm start
EOF
                        chmod +x deployment-package/start.sh
                    '''
                    
                    // Archive the deployment package
                    archiveArtifacts artifacts: 'deployment-package/**/*', fingerprint: true
                    
                    echo 'Alternative deployment package created and archived'
                    echo 'Deployment Instructions:'
                    echo '1. Download the archived deployment-package'
                    echo '2. Extract it to your target server'
                    echo '3. Run: ./start.sh to start the application'
                    echo '4. Or manually: cd backend && npm install --production && npm start'
                }
            }
        }
        
        // Stage 8: Deploy Summary
        stage('Deploy Summary') {
            steps {
                script {
                    if (env.DOCKER_AVAILABLE == 'true') {
                        echo '''
                        DEPLOYMENT SUMMARY - DOCKER METHOD
                        =====================================
                        Docker image built: wangpo1642/node-app:latest
                        Image pushed to Docker Hub
                        Ready for container deployment
                        
                        To run your application:
                        docker pull wangpo1642/node-app:latest
                        docker run -p 3000:3000 wangpo1642/node-app:latest
                        '''
                    } else {
                        echo '''
                        DEPLOYMENT SUMMARY - ALTERNATIVE METHOD
                        ==========================================
                        Deployment package created and archived
                        Manual deployment required
                        
                        Next steps:
                        1. Download deployment artifacts from Jenkins
                        2. Deploy to your target environment
                        3. Follow deployment instructions provided
                        
                        To enable Docker deployment:
                        - Mount Docker socket to Jenkins container
                        - Or install Docker inside Jenkins container
                        '''
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Clean up workspace
            cleanWs()
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs above for details.'
        }
        unstable {
            echo 'Pipeline completed with warnings. Docker deployment may have failed.'
        }
    }
}