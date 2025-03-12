# KF Monolith Clients Bank DevOps

This project contains the CI/CD configuration for the KF Monolith Clients Bank project, using Jenkins in Docker.

## 📋 Table of Contents

- [Requirements](#requirements)
- [Project Structure](#project-structure)
- [Initial Setup](#initial-setup)
- [Usage](#usage)
- [Jenkins Pipeline](#jenkins-pipeline)
- [Docker Configuration](#docker-configuration)
- [Troubleshooting](#troubleshooting)

## 🔧 Requirements

- Docker 20.10 or higher
- Docker Compose 2.0 or higher
- Git
- Java 17 (for local development)
- Maven 3.8+ (for local development)

## 📁 Project Structure

```
.
├── docker-compose.yml      # Docker Compose configuration
├── jenkins.Dockerfile      # Custom Jenkins Dockerfile
├── Jenkinsfile            # CI/CD Pipeline
├── plugins.txt            # Jenkins plugins list
└── README.md             # This file
```

## 🚀 Initial Setup

### 1. Clone the Repository

```bash
git clone https://github.com/your-username/kf-monolith-clients-bank-devops.git
cd kf-monolith-clients-bank-devops
```

### 2. Configure Environment Variables

Copy the example environment file and configure your variables:

```bash
cp .env.example .env
```

Edit the `.env` file with your settings:

```env
# Jenkins Admin Credentials
JENKINS_ADMIN_USER=admin
JENKINS_ADMIN_PASSWORD=your-secure-password

# Docker Registry Configuration
DOCKER_REGISTRY=your-docker-registry.com
DOCKER_REGISTRY_USER=your-docker-username
DOCKER_REGISTRY_PASSWORD=your-docker-password

# GitHub Configuration
GITHUB_USER=your-github-username
GITHUB_TOKEN=your-github-token
```

### 3. Configure Jenkins Pipeline

The pipeline will be automatically configured when Jenkins starts up using Jenkins Configuration as Code (JCasC). The configuration includes:

- Creation of the pipeline job
- Configuration of GitHub integration
- Setup of required credentials
- Configuration of security settings

You can modify the pipeline configuration by editing the `jenkins.yaml` file.

### 4. Start Jenkins

```bash
docker-compose up -d
```

Jenkins will be available at: http://localhost:8080/jenkins

> **Important**: Note that Jenkins is configured with a `/jenkins` context path. Always use http://localhost:8080/jenkins to access the Jenkins interface.

## 💻 Usage

### Jenkins Access

1. Navigate to http://localhost:8080/jenkins
2. Initial setup wizard is disabled by configuration
3. Use the credentials you configured in the setup step

### Jenkins Credentials Setup

1. Go to "Manage Jenkins" > "Manage Credentials"
2. Add credentials with ID: `docker-registry-credentials`
   - Kind: Username with password
   - Scope: Global
   - Username: Your Docker registry username
   - Password: Your Docker registry password
   - ID: docker-registry-credentials
   - Description: Docker Registry Credentials

## 🔄 Jenkins Pipeline

The pipeline includes the following stages:

### 1. Checkout
- Cleans the workspace
- Downloads repository code

### 2. Build
- Runs `mvn clean package -DskipTests`
- Archives generated artifacts

### 3. Test
- Runs unit tests
- Generates coverage reports
- Publishes test results

### 4. Build Docker Image
- Builds Docker image
- Only runs on main and develop branches

### 5. Deploy
- Publishes image to Docker registry
- Deploys based on environment (production/staging)
- Manages Docker tags

## 🐳 Docker Configuration

### Docker Registry

Modify the following variables in the `Jenkinsfile`:

```groovy
environment {
    DOCKER_REGISTRY = 'your-docker-registry.com'
    IMAGE_NAME = "${DOCKER_REGISTRY}/${GITHUB_REPO}"
}
```

### Included Jenkins Plugins

Essential plugins installed:
- Pipeline and GitHub Integration
- Docker Pipeline and Docker Build Step
- Credentials Binding
- Pipeline Stage View
- Pipeline Utility Steps
- Timestamper

## ⚠️ Troubleshooting

### Common Issues

1. **Cannot Access Jenkins**
   - Make sure you're using the correct URL: http://localhost:8080/jenkins
   - Check if the container is running: `docker-compose ps`

2. **Docker Permissions Error**
   ```bash
   sudo usermod -aG docker jenkins
   sudo systemctl restart jenkins
   ```

3. **Jenkins Cannot Access Docker Socket**
   ```bash
   chmod 666 /var/run/docker.sock
   ```

4. **Maven Wrapper Issues**
   ```bash
   chmod +x mvnw
   ```

## 🔐 Security

- Sensitive credentials should be stored in Jenkins Credentials
- The `.env` file should never be committed to the repository
- Use secrets for production environment

## 📝 Additional Notes

- Pipeline is configured to run automatically with GitHub webhooks
- Artifacts are preserved for 5 successful builds
- Logs are rotated every 10 builds
- Global timeout of 1 hour to prevent infinite builds
