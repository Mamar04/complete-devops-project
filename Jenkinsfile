pipeline{
    agent{
        label "jenkins-agent"
        }
        tools {
            jdk 'Java17'
            maven 'Maven3'
        }

      environment {
            APP_NAME = "complete-devops-project"
            RELEASE = "1.0.0"
            DOCKER_USER = "yhdm"
            DOCKER_PASS = 'dockerhubcred'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    
        }   
        stages{
            stage("Cleanup Workspace"){
                steps {
                    cleanWs()
                }
    
            }
        
        stage("Connection github"){
            steps {
                git branch: 'main', credentialsId: 'githubcred', url: 'https://github.com/Mamar04/complete-devops-project.git'
            }
        }
        
        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }
        }

        stage("Test Application"){
            steps {
                sh "mvn test"
            }

        }

         
        stage("Sonarqube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }

        }

        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
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

        stage('Deploy App on k8s') {
            steps {
                sshagent(['k8spwd']) {
                    sh "scp -o StrictHostKeyChecking=no deployment.yaml vagrant@10.10.10.65:/home/vagrant"
                    script {
                        try {
                            sh "ssh vagrant@10.10.10.65 kubectl create -f /home/vagrant/deployment.yml"
                        } catch(error) {
                            sleep 30 // Add a delay of 30 seconds before retrying
                            sh "ssh vagrant@10.10.10.65 kubectl create -f /home/vagrant/deployment.yml"
                        }
                    }
                }
            }
        }
     }
}   
