pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        maven 'Maven3'
        jdk 'Java17'
    }
    environment {
        APP_NAME = 'register-app-pipeline'
        RELEASE = '1.0.0'
        DOCKER_USER = 'gowdrr001'
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}:${RELEASE}"
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage("Checkout from SCM") {
            steps {
                git branch: 'main', credentialsId: 'gowdrr001', url: 'https://github.com/gowdrr001/register-app.git'
            }
        }
        stage("Build Application") {
            steps {
                sh 'mvn clean package'
            }
        }
         stage("Run Tests") {
            steps {
                sh 'mvn test'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv( credentialsId: 'jenkins-sonarqube-token' ) {
                        sh 'mvn sonar:sonar'
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }

       }
    }
}
