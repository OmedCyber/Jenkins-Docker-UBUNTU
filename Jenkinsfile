pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token') // optional: update if you're pushing images
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/OmedCyber/Jenkins-Docker-UBUNTU.git',
                        credentialsId: 'github-jenkins-token'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                echo 'Running Build Stage...'
                // Example: mvn clean install or gradle build
            }
        }

        stage('Test') {
            steps {
                echo 'Running Tests...'
                // Example: mvn test or npm test
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo 'Running Static Analysis...'
                // Example: sonar-scanner
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("jenkins-custom-image")
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression { return env.DOCKERHUB_CREDENTIALS != null }
            }
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-token') {
                        docker.image('jenkins-custom-image').push('latest')
                    }
                }
            }
        }
    }
}
