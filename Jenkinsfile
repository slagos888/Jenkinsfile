pipeline {
    agent any

    environment {
        NAMESPACE = 'jenkins'
    }

    stages {
        stage('Verificar archivos') {
            steps {
                sh '''
                    echo "Archivos en el repositorio:"
                    ls -la

                    echo ls
                
                    test -f k8s/wordpress.yaml || exit 1
                '''
            }
        }
        stage('Mostrar información') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-credentials',
                                serverUrl: 'https://192.168.49.2:8443',
                                namespace: 'jenkins']) {
                    sh '''
                        
                        kubectl get svc 
                    '''
                }
            }
        }
    }
}
