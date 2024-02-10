pipeline{
    agent{
        label "jenkins-agent"
    }
    tools {
        jdk 'Java17'
        maven 'Maven3'
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
        
    }
}
