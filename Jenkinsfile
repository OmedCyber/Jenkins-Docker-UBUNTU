pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/OmedCyber/Jenkins-Docker-UBUNTU.git',
                    credentialsId: 'github-jenkins-token' // This should match the ID you used when adding the token
            }
        }

        stage('Build') {
            steps {
                echo 'Building the project...'
                // Replace with actual build steps, for example:
                sh 'mvn clean package'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                // Replace with actual test steps
                sh 'mvn test'
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo 'Running static code analysis...'
                // Example: SonarQube integration, etc.
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh 'docker build -t my-jenkins-app .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing Docker image to Docker Hub...'
                // Make sure Docker credentials are configured
                sh '''
                echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                docker push my-jenkins-app
                '''
            }
        }
    }
}
