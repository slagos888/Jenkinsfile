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
                    // Espera a que el rollout de WordPress se complete con éxito (asumiendo que tu deployment se llama 'wordpress')
                    sh '''
                        echo "Esperando que los Pods estén listos..."
                        kubectl rollout status deployment/wordpress -n ${NAMESPACE} --timeout=90s
                        
                        echo "Recursos actuales en el namespace ${NAMESPACE}:"
                        kubectl get all -n ${NAMESPACE}
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
            echo 'El despliegue falló. Revisa los logs de Kubernetes.'
        }
    }
}
