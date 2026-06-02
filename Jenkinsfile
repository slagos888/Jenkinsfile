pipeline {
    agent {
        kubernetes {
            // Usamos la ServiceAccount que creamos con permisos para Deployments, Pods y PVCs
            serviceAccountName 'jenkins-deploy'
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["cat"]
    tty: true
'''
        }
    }

    stages {
        stage('1. Descargar Código') {
            steps {
                checkout scm
            }
        }

        stage('2. Preparar Entorno') {
            steps {
                container('kubectl') {
                    echo "Asegurando que el namespace 'wordpress-prod' exista..."
                    // Eliminamos el 'withKubeConfig' ya que el contenedor hereda los permisos automáticamente
                    sh 'kubectl create namespace wordpress-prod --dry-run=client -o yaml | kubectl apply -f -'
                }
            }
        }

        stage('3. Desplegar Base de Datos') {
            steps {
                container('kubectl') {
                    echo "Desplegando MySQL..."
                    // Asegúrate de pasar el namespace correcto si tus manifiestos no lo especifican dentro
                    sh 'kubectl apply -f k8s/mysql-deployment.yaml -n wordpress-prod'
                }
            }
        }

        stage('4. Desplegar WordPress') {
            steps {
                container('kubectl') {
                    echo "Desplegando WordPress..."
                    sh 'kubectl apply -f k8s/wordpress-deployment.yaml -n wordpress-prod'
                }
            }
        }

        stage('5. Verificar Despliegue') {
            steps {
                container('kubectl') {
                    echo "Validando estado de los Pods..."
                    sh 'kubectl get pods -n wordpress-prod'
                }
            }
        }
    }

    post {
        failure {
            echo 'El despliegue falló. Revisa los logs de Jenkins o el estado del clúster.'
        }
        success {
            echo '¡Despliegue de WordPress y MySQL completado con éxito!'
        }
    }
}
