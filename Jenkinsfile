pipeline {
    agent any

    environment {
        NAMESPACE = 'jenkins'
        YAML_PATH = 'k8s/wordpress.yaml'
    }

    stages {
        stage('Verificar archivos') {
            steps {
                sh '''
                    echo "Archivos en el repositorio:"
                    ls -la
                    
                    # Valida que el archivo exista antes de continuar
                    test -f ${YAML_PATH} || { echo "Error: ${YAML_PATH} no encontrado"; exit 1; }
                '''
            }
        }

        stage('Mostrar información del Clúster') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-credentials',
                                serverUrl: 'https://192.168.49.2:8443',
                                namespace: "${NAMESPACE}"]) {
                    sh 'kubectl get svc'
                }
            }
        }

        stage('Desplegar WordPress') {
            steps {
                withKubeConfig([credentialsId: 'kubeconfig-credentials',
                                serverUrl: 'https://192.168.49.2:8443',
                                namespace: "${NAMESPACE}"]) {
                    sh '''
                        echo "Aplicando manifiestos de Kubernetes..."
                        kubectl apply -f ${YAML_PATH} -n ${NAMESPACE}
                    '''
                }
            }
        }

        stage('Verificar Despliegue') {
    steps {
        withKubeConfig([credentialsId: 'kubeconfig-credentials',
                        serverUrl: 'https://192.168.49.2:8443',
                        namespace: "${NAMESPACE}"]) {
            sh '''
                echo "=== Estado actual de los Pods ==="
                kubectl get pods -n ${NAMESPACE}

                echo "=== Estado de los Almacenamientos (PVC) ==="
                kubectl get pvc -n ${NAMESPACE}

                echo "=== Esperando por el Deployment (intento de rollout) ==="
                # Intentamos el rollout pero capturamos si falla para que no rompa el script antes de ver los eventos
                kubectl rollout status deployment/wordpress -n ${NAMESPACE} --timeout=40s || true

                echo "=== Descripción de los Pods (Aquí verás los errores reales) ==="
                kubectl describe pods -n ${NAMESPACE}
                
                echo "=== Eventos recientes del clúster ==="
                kubectl get events -n ${NAMESPACE} --sort-by='.metadata.creationTimestamp' | tail -n 20
            '''
        }
    }
}
    
    post {
        always {
            echo 'Pipeline finalizado.'
        }
        failure {
            echo 'El despliegue falló. Revisa los logs de Kubernetes.'
        }
    }
}
