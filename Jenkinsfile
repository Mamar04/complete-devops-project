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

        stage("Sonarqube Analysis") {
            steps {
                withSonarQubeEnv(installationName: 'sonarqube-scanner', credentialsId: 'sonar-token') {
                    sh "mvn sonar:sonar"
                }
            }
        }

        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
            }
        }
        
        stage("Build & Push Docker Image") {
                
            steps {
        script {
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhubcred') {
                def customImage = docker.build("${IMAGE_NAME}")
                customImage.push("${IMAGE_TAG}")
                customImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy to kubernets'){
            steps{
                script{
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8sclus', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                       sh 'kubectl apply -f kaniko-builder.yaml'
                  }
                }
            }
        }
    
    }
}
