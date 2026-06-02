pipeline {
    agent any

    environment {
        NAMESPACE = 'wordpress-prod-jenkins'
    }

    stages {
        stage('Verificar archivos') {
            steps {
                sh '''
                    echo "Archivos en el repositorio:"
                    ls -la

                    
                    test -f K8s/wordpress.yaml || exit 1
                '''
            }
        }
}
