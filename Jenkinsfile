pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token') // optional
        SONAR_TOKEN = credentials('sonarqube-token') // 🔐 must match your SonarQube token ID in Jenkins
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
                echo '🔧 Running Build Stage...'
                // Example: mvn clean install
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running Tests...'
                // Example: mvn test
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo '🔍 Running Static Code Analysis with SonarQube...'
                withSonarQubeEnv('SonarQube') {
                    sh 'which mvn' // ✅ confirms Maven is available
                    sh '''
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=JenkinsDockerFinal \
                          -Dsonar.host.url=http://localhost:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
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
