pipeline {
    agent any
    
    triggers {
        // Configure GitHub webhook trigger
        githubPush()
    }
    
    options {
        // Set timeout for the entire pipeline
        timeout(time: 1, unit: 'HOURS')
        // Preserve artifacts and stashes from the last successful build
        preserveStashes(buildCount: 5)
        // Discard old builds
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    environment {
        // Define environment variables
        GITHUB_REPO = 'kf-monolith-clients-bank'
        DEPLOY_ENV = "${env.BRANCH_NAME == 'main' ? 'production' : 'staging'}"
        DOCKER_REGISTRY = 'registry.example.com'  // Replace with your registry
        IMAGE_NAME = "${DOCKER_REGISTRY}/${GITHUB_REPO}"
        IMAGE_TAG = "${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                // Clean workspace before checkout
                cleanWs()
                checkout scm
                echo "Code checked out from branch: ${env.BRANCH_NAME}"
            }
        }
        
        stage('Build') {
            steps {
                echo "Building application..."
                sh '''
                    # Ensure Maven wrapper is executable
                    chmod +x ./mvnw
                    
                    # Build without running tests
                    ./mvnw clean package -DskipTests
                '''
                
                // Archive the artifacts
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
            post {
                success {
                    echo "Build completed successfully!"
                }
                failure {
                    echo "Build failed!"
                }
            }
        }
        
        stage('Test') {
            steps {
                echo "Running tests..."
                sh '''
                    # Run tests
                    ./mvnw test
                '''
            }
            post {
                always {
                    // Publish test results
                    junit '**/target/surefire-reports/*.xml'
                    
                    // Optional: Publish code coverage
                    jacoco(
                        execPattern: 'target/*.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java',
                        exclusionPattern: 'src/test*'
                    )
                }
                success {
                    echo "All tests passed!"
                }
                failure {
                    echo "Tests failed!"
                }
            }
        }
        
        stage('Build Docker Image') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    // Build Docker image
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
            }
        }
        
        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    echo "Deploying to ${DEPLOY_ENV} environment..."
                    
                    // Login to Docker registry (using credentials)
                    withCredentials([usernamePassword(
                        credentialsId: 'docker-registry-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )]) {
                        sh '''
                            echo $DOCKER_PASSWORD | docker login ${DOCKER_REGISTRY} -u ${DOCKER_USER} --password-stdin
                            
                            # Push Docker image to registry
                            docker push ${IMAGE_NAME}:${IMAGE_TAG}
                            
                            # Tag as latest if on main branch
                            if [ "${DEPLOY_ENV}" = "production" ]; then
                                docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
                                docker push ${IMAGE_NAME}:latest
                            fi
                        '''
                    }
                    
                    // Dummy deployment steps based on environment
                    if (env.DEPLOY_ENV == 'production') {
                        echo "Deploying to production environment..."
                        // Add production deployment steps here
                    } else {
                        echo "Deploying to staging environment..."
                        // Add staging deployment steps here
                    }
                }
            }
            post {
                success {
                    echo "Deployment to ${DEPLOY_ENV} completed successfully!"
                }
                failure {
                    echo "Deployment to ${DEPLOY_ENV} failed!"
                }
            }
        }
    }
    
    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
            // Send email notification on failure
            mail to: 'team@example.com',
                 subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
                 body: "Pipeline failed: ${env.BUILD_URL}"
        }
        always {
            // Clean up Docker images
            sh '''
                docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                docker rmi ${IMAGE_NAME}:latest || true
            '''
            cleanWs()
        }
    }
}

