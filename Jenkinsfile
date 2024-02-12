pipeline {
    agent any
    
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }

    environment {
        APP_NAME = "complete-devops-project"
        RELEASE = "1.0.0"
        DOCKER_USER = "yhdm"
        DOCKER_PASS = credentials('dockerhubcred')
        IMAGE_NAME = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        KUBE_CONFIG = credentials('b72377b4-c4e0-4053-8f92-de072945e679')
     
    }
    stages {
        stage("Cleanup Workspace") {
            steps {
                cleanWs()
            }
        }
        
        stage("Connection github") {
            steps {
                git branch: 'main', credentialsId: 'Gitcredentials', url: 'https://github.com/Mamar04/complete-devops-project.git'
            }
        }
        
        stage("Build Application") {
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application") {
            steps {
                sh "mvn test"
            }
        }
            }
        }
 
