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
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
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
        stage("Build Docker Image") {
            steps {
                script {
                    docker.withRegistry('', 'gowdrr001') {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("${RELEASE}")
                        docker_image.push('latest')
                    }
                }
            }
        }
       stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user admin:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'http://192.168.60.135:8080/job/gitops-project/build?token=gitops-token'"
                }
            }
       }        
    }
}
