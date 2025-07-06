pipeline {
    agent any

    environment {
        NPM_CONFIG_CACHE = "${WORKSPACE}/.npm"
        dockerImagePrefix = "us-west1-docker.pkg.dev/lab-agibiz/docker-repository"
        registry = "https://us-west1-docker.pkg.dev"
        registryCredentials = "gcp-registry"
    }

    stages {
        stage("saludo a usuario") {
            steps {
                sh 'echo "Comenzando mi pipeline, saludos desde Jenkins."'
            }
        }

        stage("salida de los saludos a usuario") {
            steps {
                sh 'echo "Finalizando saludo, vamos al build."'
            }
        }

        stage("proceso de build y test") {
            agent {
                docker {
                    image 'node:22'
                    reuseNode true
                }
            }

            stages {
                stage("instalacion de dependencias") {
                    steps {
                        sh 'npm ci'
                    }
                }

                stage("ejecucion de pruebas") {
                    steps {
                        sh 'npm run test:cov'
                    }
                }

                stage("construccion de la aplicacion") {
                    steps {
                        sh 'npm run build'
                    }
                }
            }
        }

        stage("build y push de imagen docker") {
            steps {
                script {
                    def imageName = "javierbarahona/backend-nest-test-jbs"
                    def finalImageName = "backend-nest-test-jbs"
                    docker.withRegistry("${registry}", registryCredentials) {
                        sh "docker build -t ${imageName} ."

                        sh "docker tag ${imageName} ${dockerImagePrefix}/${finalImageName}:latest"
                        sh "docker tag ${imageName} ${dockerImagePrefix}/${finalImageName}:${BUILD_NUMBER}"
                        sh "docker push ${dockerImagePrefix}/${finalImageName}:latest"
                        sh "docker push ${dockerImagePrefix}/${finalImageName}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage ("actualizacion de kubernetes") {
            agent {
                docker {
                    image 'alpine/k8s:1.30.2'
                    reuseNode true
                }
            }
            steps {
                withKubeConfig([credentialsId: 'gcp-kubeconfig']) {
                    sh "kubectl -n lab-test-jbs set image deployments/backend-nest-test-jbs backend-nest-test-jbs=${dockerImagePrefix}/backend-nest-test-jbs:${BUILD_NUMBER}"
                }
            }
        }

    }
}
