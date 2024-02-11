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
                git branch: 'main', credentialsId: 'githubcred', url: 'https://github.com/Mamar04/complete-devops-project.git'
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
                    docker.withRegistry([credentialsId: 'dockerhubcred']) {
                        def docker_image = docker.build("${IMAGE_NAME}")
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                            }
                }
            }
        }

        stage('Build & Push k8scluster') {
            steps {
                script {
                    podTemplate(
                        label: 'mypod',
                        containers: [
                            containerTemplate(
                                name: 'demoapp',
                                image: 'yhdm/complete-devops-project:latest',
                                command: ['/busybox/sh'],
                                ttyEnabled: true
                            )
                        ],
                        volumes: [
                            secretVolume(secretName: 'docker-credentials', mountPath: '/root/.docker')
                        ]
                    ) {
                        node('mypod') {
                            container('demoapp') {
                                sh '''
                                    #!/busybox/sh
                                    /kaniko/executor --dockerfile `pwd`/Dockerfile --context `pwd` --destination=${IMAGE_NAME}:${IMAGE_TAG} --destination=${IMAGE_NAME}:latest
                                '''
                            }
                        }
                    }
                }
            }
        }
    }
}
