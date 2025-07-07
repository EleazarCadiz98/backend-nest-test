pipeline {

    agent any

    environment {
        NPM_CONFIG_CACHE= "${WORKSPACE}/.npm"
        DOCKER_REPOSITORY_NAME= "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        DOCKER_REGISTRY= "https://us-west1-docker.pkg.dev"
        DOCKER_REGISTRY_CREDENTIALS= "gcp-registry-ele"  
        DOCKER_IMAGE_NAME = "backend-nest-test-ele"
        DOCKER_CONTAINER_NAME_IN_KUBERNETES = "backend-nest-test-elecontainer"
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
                        sh "docker build -t ${DOCKER_IMAGE_NAME} ." 
                        sh "docker tag ${DOCKER_IMAGE_NAME} ${DOCKER_REPOSITORY_NAME}/${DOCKER_IMAGE_NAME}"
                        sh "docker tag ${DOCKER_IMAGE_NAME} ${DOCKER_REPOSITORY_NAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                        sh "docker push ${DOCKER_REPOSITORY_NAME}/${DOCKER_IMAGE_NAME}" 
                        sh "docker push ${DOCKER_REPOSITORY_NAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}" 
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
                    sh "kubectl -n lab-ele set image deployments/${DOCKER_IMAGE_NAME} ${DOCKER_CONTAINER_NAME_IN_KUBERNETES}=${DOCKER_REPOSITORY_NAME}/${DOCKER_IMAGE_NAME}:${BUILD_NUMBER}"
                }
            }
        }
    }
}