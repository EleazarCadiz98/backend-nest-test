pipeline {

    agent any

    environment {
        NPM_CONFIG_CACHE= "${WORKSPACE}/.npm"
        DOCKER_IMAGE_NAME= "us-west1-docker.pkg.dev/lab-agibiz/docker-repository" // Repositorio en GCP
        DOCKER_REGISTRY= "https://us-west1-docker.pkg.dev" // Usuario ("registry") de GCP
        DOCKER_REGISTRY_CREDENTIALS= "gcp-registry-ele"  // Credenciales ("REGISTRY") de GCP
    }
    stages {
        stage ("1 - Proceso de Build & Test"){
            agent {
                docker {
                    image 'node:22'
                    reuseNode true 
                }
            }
            stages {
                stage ("[1/3] - Instalación de Dependencias") {
                    steps {
                        sh 'npm ci'
                    }
                }
                stage ("[2/3] - Ejecución de Pruebas") {
                    steps {
                        sh 'npm run test:cov' 
                    }
                }
                stage ("[3/3] - Construcción de la aplicación") {
                    steps {
                        sh 'npm run build' 
                    }
                }
            }
        }
        stage ("2 - Build y Push de imagen Docker"){
            steps {
                script {
                    docker.withRegistry("${DOCKER_REGISTRY}", DOCKER_REGISTRY_CREDENTIALS){
                        sh "docker build -t backend-nest-test-ele ." 
                        sh "docker tag backend-nest-test-ele ${DOCKER_IMAGE_NAME}/backend-nest-test-ele"
                        sh "docker tag backend-nest-test-ele ${DOCKER_IMAGE_NAME}/backend-nest-test-ele:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_IMAGE_NAME}/backend-nest-test-ele" 
                        sh "docker push ${DOCKER_IMAGE_NAME}/backend-nest-test-ele:${BUILD_NUMBER}" 
                    }
                }
                
            }
        }
        stage ("3 - Actualizacion de kubernetes"){
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']){
                    sh "kubectl -n lab-ele set image deployments/backend-nest-test-ele backend-nest-test-ele=${DOCKER_IMAGE_NAME}/backend-nest-test-ele:${BUILD_NUMBER}"
                }
            }
        }
    }
}