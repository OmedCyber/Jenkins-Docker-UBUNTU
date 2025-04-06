pipeline { 
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token')     // 🔐 Docker Hub credentials
        SONAR_TOKEN = credentials('sonarqube-token')               // 🔐 SonarQube token
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir() // 🧼 Clean up before build
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
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running Tests...'
                sh 'mvn test'
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo '🔍 Running Static Code Analysis with SonarQube... 🎯'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn clean verify sonar:sonar \
                          -Dsonar.projectKey=JenkinsDockerFinal \
                          -Dsonar.host.url=http://sonar:9000 \
                          -Dsonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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
