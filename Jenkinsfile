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

                        echo "=== Intento de Rollout (Timeout corto) ==="
                        kubectl rollout status deployment/wordpress -n ${NAMESPACE} --timeout=20s || true

                        echo "=== Descripción de los Pods ==="
                        kubectl describe pods -n ${NAMESPACE}
                        
                        echo "=== Eventos recientes ==="
                        kubectl get events -n ${NAMESPACE} --sort-by='.metadata.creationTimestamp' | tail -n 15
                    '''
                }
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finalizado.'
        }
        failure {
            echo 'El despliegue falló. Revisa los detalles impresos en la etapa de Verificación.'
        }
    }
}
