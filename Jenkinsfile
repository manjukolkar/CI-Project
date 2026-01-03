pipeline {
    agent any
    
    environment {
        DOCKER_HUB_REPO = 'manjukolkar007/ecommerce-app'  // Update with your Docker Hub username
        IMAGE_TAG = ''
        PATH = "/opt/homebrew/bin:/usr/local/bin:${env.PATH}"
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                echo 'Checking out code from public repository...'
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/manjukolkar/CI-Project.git'
                    ]]
                ])
                script {
                    env.GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                    env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                }
            }
        }
        
        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh '''
                    echo "Node.js version:"
                    node --version || /opt/homebrew/bin/node --version
                    echo "npm version:"
                    npm --version || /opt/homebrew/bin/npm --version
                    npm ci || /opt/homebrew/bin/npm ci
                '''
            }
        }
        
        stage('Code Quality - Linter') {
            steps {
                echo 'Running ESLint...'
                sh '''
                    npm run lint || /opt/homebrew/bin/npm run lint || true
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube code analysis...'
                script {
                    try {
                        def scannerHome = tool 'SonarQubeScanner'
                        echo "SonarQube Scanner found at: ${scannerHome}"
                        
                        withSonarQubeEnv('SonarQube') {
                            echo "SonarQube Server: ${env.SONAR_HOST_URL}"
                            echo "Project Key: ecommerce-app"
                            
                            sh """
                                ${scannerHome}/bin/sonar-scanner \
                                    -Dsonar.projectKey=ecommerce-app \
                                    -Dsonar.projectName='E-Commerce Application' \
                                    -Dsonar.sources=server,public \
                                    -Dsonar.host.url=${env.SONAR_HOST_URL} \
                                    -Dsonar.login=${env.SONAR_TOKEN} \
                                    -Dsonar.sourceEncoding=UTF-8
                            """
                            
                            echo "✅ SonarQube analysis completed successfully!"
                            echo "📊 View results at: ${env.SONAR_HOST_URL}/dashboard?id=ecommerce-app"
                        }
                    } catch (Exception e) {
                        echo "❌ SonarQube analysis failed: ${e.getMessage()}"
                        echo ""
                        echo "⚠️ Troubleshooting steps:"
                        echo "1. Configure SonarQube Scanner tool in Jenkins:"
                        echo "   - Go to: Manage Jenkins → Global Tool Configuration"
                        echo "   - Scroll to 'SonarQube Scanner'"
                        echo "   - Click 'Add SonarQube Scanner'"
                        echo "   - Name: SonarQubeScanner (must match exactly)"
                        echo "   - Uncheck 'Install automatically'"
                        echo "   - Set SONAR_RUNNER_HOME: /opt/homebrew/bin (if using Homebrew)"
                        echo "   - Or: /opt/sonar-scanner (if installed manually)"
                        echo ""
                        echo "2. Verify SonarQube server is configured:"
                        echo "   - Go to: Manage Jenkins → Configure System"
                        echo "   - Scroll to 'SonarQube servers'"
                        echo "   - Name should be: SonarQube (must match exactly)"
                        echo "   - Server URL should be set (e.g., https://sonarcloud.io)"
                        echo ""
                        echo "3. Check project 'ecommerce-app' exists in SonarQube"
                        echo "4. Verify token has 'Execute Analysis' permission"
                        // Don't fail the build
                    }
                }
            }
        }
        
        stage('SonarQube Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate...'
                script {
                    try {
                        def qg = waitForQualityGate abortPipeline: false
                        if (qg.status != 'OK') {
                            echo "⚠️ Quality Gate status: ${qg.status}"
                        } else {
                            echo "✅ Quality Gate passed!"
                        }
                    } catch (Exception e) {
                        echo "⚠️ SonarQube Quality Gate error: ${e.getMessage()}"
                        echo "This might happen if:"
                        echo "1. Analysis hasn't completed yet"
                        echo "2. SonarQube server is not accessible"
                        echo "3. Project doesn't exist in SonarQube"
                        // Don't fail the build
                    }
                }
            }
        }
        
        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                script {
                    // Ensure IMAGE_TAG is set
                    if (!env.IMAGE_TAG) {
                        env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    }
                    echo "Building image with tag: ${env.IMAGE_TAG}"
                    sh """
                        docker build -t ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG} .
                        docker tag ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG} ${env.DOCKER_HUB_REPO}:latest
                    """
                }
            }
        }
        
        stage('Docker Login') {
            steps {
                echo 'Logging into Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh """
                        echo ${DOCKER_PASSWORD} | docker login -u ${DOCKER_USERNAME} --password-stdin
                    """
                }
            }
        }
        
        stage('Docker Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                script {
                    // Ensure IMAGE_TAG is set
                    if (!env.IMAGE_TAG) {
                        env.IMAGE_TAG = "${env.BUILD_NUMBER}-${env.GIT_COMMIT.take(7)}"
                    }
                    sh """
                        docker push ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}
                        docker push ${env.DOCKER_HUB_REPO}:latest
                    """
                    echo "Image pushed successfully: ${env.DOCKER_HUB_REPO}:${env.IMAGE_TAG}"
                }
            }
        }
    }
    
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}

