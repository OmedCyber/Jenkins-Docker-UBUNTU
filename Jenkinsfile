pipeline {
    agent any

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-token')
        SONAR_TOKEN = credentials('sonarqube-token')
        SONARQUBE_SERVER = 'http://sonar:9000'
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
                echo 'Running Build Stage with Maven + Java 17...'
                script {
                    docker.image('maven:3.8.7-eclipse-temurin-17').inside {
                        sh 'mvn clean compile'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running Tests with Maven + Java 11...'
                script {
                    docker.image('maven:3.8.7-eclipse-temurin-11').inside {
                        sh 'mvn test'
                    }
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                echo 'Running Static Code Analysis with SonarQube + Java 8...'
                script {
                    docker.image('maven:3.8.7-eclipse-temurin-8').inside('--network=ci_network') {
                        withSonarQubeEnv('SonarQube') {
                            sh """
                                mvn verify sonar:sonar \
                                  -Dsonar.projectKey=JenkinsDockerFinal \
                                  -Dsonar.host.url=${SONARQUBE_SERVER} \
                                  -Dsonar.login=${SONAR_TOKEN} \
                                  -Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml
                            """
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
                echo 'Building Docker Image...'
                script {
                    docker.build("roguemain12/math-utils")
                }
            }
        }

        stage('Push to Docker Hub') {
            when {
                expression { return env.DOCKERHUB_CREDENTIALS != null }
            }
            steps {
                echo 'Pushing Docker Image to Docker Hub...'
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-token') {
                        docker.image('roguemain12/math-utils').push('latest')
                    }
                }
            }
        }
    }
}
