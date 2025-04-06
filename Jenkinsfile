pipeline {
    agent {
        docker {
            image 'maven:3.8.7-eclipse-temurin-17' // Build with Java 17
            args '-v /root/.m2:/root/.m2'
        }
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token')
        SONAR_TOKEN = credentials('sonarqube-token')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                deleteDir()
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
                echo '🔧 Running Build Stage with Java 17...'
                script {
                    docker.image('openjdk:17-jdk').inside {
                        sh 'mvn clean compile'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running Tests with Java 11...'
                script {
                    docker.image('openjdk:11-jdk').inside {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo '🔍 Running Static Code Analysis with SonarQube (Java 8)...'
                script {
                    docker.image('openjdk:8-jdk').inside {
                        withSonarQubeEnv('SonarQube') {
                            sh '''
                                mvn verify sonar:sonar \
                                  -Dsonar.projectKey=JenkinsDockerFinal \
                                  -Dsonar.host.url=http://sonar:9000 \
                                  -Dsonar.login=$SONAR_TOKEN \
                                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            '''
                        }
                    }
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
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
