pipeline {
    agent any

    environment {
        NAMESPACE = 'default'
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
    }
}
