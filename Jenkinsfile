pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'http://sonar:9000'
        DOCKER_IMAGE = 'roguemain12/math-utils:latest'
    }

    stages {
        stage('Build') {
            steps {
                script {
                    docker.image('openjdk:17-jdk').inside {
                        sh 'mvn clean package -DskipTests=true'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    docker.image('openjdk:11-jdk').inside {
                        sh 'mvn test'
                    }
                }
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Static Code Analysis') {
            steps {
                script {
                    docker.image('openjdk:8-jdk').inside {
                        sh 'mvn sonar:sonar -Dsonar.host.url=${SONARQUBE_SERVER}'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                }
            }
        }
    }
}
