pipeline {
    agent { label 'Jenkins-Agent' }
    tools {
        maven 'Maven3'
        jdk 'Java17'
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
    }
}
