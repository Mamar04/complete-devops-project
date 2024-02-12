stage('Build & Push k8scluster') {
    steps {
        script {
            podTemplate(
                label: 'k8scluster',
                containers: [
                    containerTemplate(
                        name: 'demoapp',
                        image: 'yhdm/complete-devops-project:latest',
                        command: '/busybox/sh',
                        ttyEnabled: true
                    )
                ],
                volumes: [
                    secretVolume(secretName: 'docker-credentials', mountPath: '/root/.docker')
                ]
            ) {
                node('k8scluster') {
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
