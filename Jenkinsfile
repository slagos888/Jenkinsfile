pipeline {
    agent any

    environment {
        // Definimos el ID de las credenciales de Kubernetes configuradas en Jenkins
        KUBE_CREDENTIALS_ID = 'kubeconfig-credentials'
        // Definimos el espacio de nombres donde se desplegará WordPress
        NAMESPACE           = 'wordpress-prod'
    }

    stages {
        stage('1. Descargar Código') {
            steps {
                // Descarga los manifiestos de Kubernetes desde tu repositorio de Git
                checkout scm
            }
        }

        stage('2. Preparar Entorno') {
            steps {
                script {
                    echo "Asegurando que el namespace '${NAMESPACE}' exista..."
                    // Usamos la herramienta conKubeconfig para ejecutar comandos con acceso al clúster
                    withKubeConfig([credentialsId: String.valueOf(env.KUBE_CREDENTIALS_ID)]) {
                        sh "kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -"
                    }
                }
            }
        }

        stage('3. Desplegar Base de Datos') {
            steps {
                script {
                    echo "Desplegando almacenamiento y base de datos MySQL..."
                    withKubeConfig([credentialsId: String.valueOf(env.KUBE_CREDENTIALS_ID)]) {
                        // Se asume que tienes un archivo consolidado o una carpeta para la BD
                        // Por ejemplo: k8s/mysql-deployment.yaml
                        sh "kubectl apply -f k8s/mysql-deployment.yaml -n ${NAMESPACE}"
                        
                        echo "Esperando que MySQL esté listo..."
                        sh "kubectl rollout status deployment/wordpress-mysql -n ${NAMESPACE} --timeout=90s"
                    }
                }
            }
        }

        stage('4. Desplegar WordPress') {
            steps {
                script {
                    echo "Desplegando WordPress..."
                    withKubeConfig([credentialsId: String.valueOf(env.KUBE_CREDENTIALS_ID)]) {
                        // Se aplica el despliegue de la aplicación de WordPress
                        // Por ejemplo: k8s/wordpress-deployment.yaml
                        sh "kubectl apply -f k8s/wordpress-deployment.yaml -n ${NAMESPACE}"
                        
                        echo "Esperando que WordPress esté listo..."
                        sh "kubectl rollout status deployment/wordpress -n ${NAMESPACE} --timeout=90s"
                    }
                }
            }
        }

        stage('5. Verificar Despliegue') {
            steps {
                script {
                    withKubeConfig([credentialsId: String.valueOf(env.KUBE_CREDENTIALS_ID)]) {
                        echo "=== Estado de los Pods ==="
                        sh "kubectl get pods -n ${NAMESPACE}"
                        echo "=== Servicios Disponibles ==="
                        sh "kubectl get svc -n ${NAMESPACE}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "¡El pipeline finalizó con éxito! WordPress ha sido actualizado en Kubernetes."
        }
        failure {
            echo "El despliegue falló. Revisa los logs de Jenkins o el estado del clúster."
        }
    }
}
