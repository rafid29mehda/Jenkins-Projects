# Jenkins-Projects

Running Jenkins in a Docker container and creating a Jenkinsfile for a complete CI/CD pipeline involves several steps. Here's a step-by-step guide:

### Step 1: Run Jenkins in a Docker Container

1. Pull the Jenkins Docker image:

   ```bash
   docker pull jenkins/jenkins:lts
   ```

2. Run Jenkins as a Docker container:

   ```bash
   docker run -d -p 8080:8080 -p 50000:50000 --name jenkins -v jenkins_home:/var/jenkins_home jenkins/jenkins:lts
   ```

   This command runs Jenkins in detached mode (`-d`), maps host ports to container ports (`-p`), sets the container name, and uses a volume (`-v`) to persist Jenkins data.

3. Access Jenkins by opening a web browser and navigating to `http://localhost:8080`. Retrieve the initial admin password from the Jenkins container:

   ```bash
   docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
   ```

   Copy the password and paste it into the Jenkins setup wizard.

### Step 2: Install Required Jenkins Plugins

Install the necessary plugins for Docker and Git integration:

1. Go to Jenkins > Manage Jenkins > Manage Plugins.
2. In the "Available" tab, search and install the following plugins:
   - Docker
   - Git
   - Pipeline

### Step 3: Create a Jenkinsfile

Create a Jenkinsfile at the root of your project. This file defines the pipeline steps:

```groovy
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    // Your build steps here
                    sh 'docker build -t your-image-name .'
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    // Your push steps here
                    withCredentials([usernamePassword(credentialsId: 'your-docker-credentials-id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
                        sh 'docker push your-image-name'
                    }
                }
            }
        }

        stage('Run') {
            steps {
                script {
                    // Your run steps here
                    sh 'docker run -d your-image-name'
                }
            }
        }
    }

    post {
        success {
            // Your success steps here
        }
        failure {
            // Your failure steps here
        }
    }
}
```

Replace placeholders like `your-image-name` with your actual image name. This pipeline checks out the code, builds a Docker image, pushes it to a registry, runs the container, and includes post-actions for success and failure.

### Step 4: Set Up GitHub Webhook

1. In your GitHub repository, go to `Settings` > `Webhooks` > `Add webhook`.
2. Set the Payload URL to `http://your-jenkins-server/github-webhook/`.
3. Content type should be `application/json`.
4. Set the webhook events based on your requirements (e.g., push events).

Now, every push to your GitHub repository triggers the Jenkins pipeline.

Remember to adapt the Jenkinsfile and configurations to match your specific project and requirements.
