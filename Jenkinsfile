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
                // Example build commands - adjust based on your project
                sh '''
                    # Build commands go here
                    echo "Running build process..."
                    # Example: 
                    # mvn clean package -DskipTests
                    # or
                    # docker build -t kf-monolith-clients-bank:${BUILD_NUMBER} .
                '''
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
                // Example test commands
                sh '''
                    # Test commands go here
                    echo "Running test suite..."
                    # Example: 
                    # mvn test
                    # or
                    # npm test
                '''
            }
            post {
                always {
                    // Publish test results if available
                    echo "Publishing test results..."
                    // Example: junit 'target/surefire-reports/*.xml'
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
                echo "Deploying to ${DEPLOY_ENV} environment..."
                // Example deployment commands
                sh '''
                    # Deployment commands go here
                    echo "Deploying application..."
                    # Example:
                    # if [ "${DEPLOY_ENV}" = "production" ]; then
                    #   # Production deployment steps
                    # else
                    #   # Staging deployment steps
                    # fi
                '''
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
            // mail to: 'team@example.com',
            //      subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
            //      body: "Pipeline failed: ${env.BUILD_URL}"
        }
        always {
            // Clean up workspace
            cleanWs()
        }
    }
}

