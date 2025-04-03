pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token') // Optional
        SONAR_TOKEN = credentials('sonarqube-token')           // 🔐 Must match ID in Jenkins
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir() // 🧼 Ensure fresh workspace
            }
        }

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
                echo '🔍 Running Static Code Analysis with SonarQube... 🎯'
                withSonarQubeEnv('SonarQube') {
                    sh 'which mvn' // ✅ Verifies Maven exists
                    sh '''
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=JenkinsDockerFinal \
                          -Dsonar.host.url=http://sonar:9000 \
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
