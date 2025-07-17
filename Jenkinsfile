def app

// pipeline {
//     agent any
//     environment {
//         ENV_TYPE = "production"
//         PORT = 3947
//         NAMESPACE = "ironsamurai-site"
//         REGISTRY_HOSTNAME = "maksimminakov42"
//         REGISTRY = "registry.hub.docker.com"
//         PROJECT = "iron-samurai"
//         DEPLOYMENT_NAME = "iron-samurai-deployment"
//         IMAGE_NAME = "${env.BUILD_ID}_${env.ENV_TYPE}_${env.GIT_COMMIT}"
//         DOCKER_BUILD_NAME = "${env.REGISTRY_HOSTNAME}/${env.PROJECT}:${env.IMAGE_NAME}"
//     }
//
//     stages {
//         stage('Clone repository') {
//             steps {
//                 checkout scm
//             }
//         }
//         stage('Unit tests') {
//              steps {
//                 echo "Preparing started..."
//                   script {
//                       sh '''
//                          export NVM_DIR="$HOME/.nvm"
//                          [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
//                          nvm use --lts
//                          yarn install
//                          yarn test
//                       '''
//                   }
//              }
//         }
//         stage('Build docker image') {
//             steps {
//                 echo "Build image started..."
//                     script {
//                         app = docker.build("${env.DOCKER_BUILD_NAME}")
//                     }
//                 echo "Build image finished..."
//             }
//         }
//         stage('Push docker image') {
//              steps {
//                  echo "Push image started..."
//                      script {
//                           docker.withRegistry("https://${env.REGISTRY}", 'ironsamurai-site') {
//                             app.push("${env.IMAGE_NAME}")
//                         }
//                      }
//                  echo "Push image finished..."
//              }
//        }
//        stage('Delete image local') {
//              steps {
//                  script {
//                     sh "docker rmi -f ${env.DOCKER_BUILD_NAME}"
//                  }
//              }
//         }
//         stage('Preparing deployment') {
//              steps {
//                  echo "Preparing started..."
//                      sh 'ls -ltr'
//                      sh 'pwd'
//                      sh "chmod +x preparingDeploy.sh"
//                      sh "./preparingDeploy.sh ${env.REGISTRY_HOSTNAME} ${env.PROJECT} ${env.IMAGE_NAME} ${env.DEPLOYMENT_NAME} ${env.PORT} ${env.NAMESPACE}"
//                      sh "cat deployment.yaml"
//              }
//
//         }
//         stage('Deploy to Kubernetes') {
//              steps {
//                  withKubeConfig([credentialsId: 'prod-kubernetes']) {
//                     sh 'kubectl apply -f deployment.yaml'
//                     sh "kubectl rollout status deployment/${env.DEPLOYMENT_NAME} --namespace=${env.NAMESPACE}"
//                     sh "kubectl get services -o wide"
//                  }
//              }
//         }
//     }
// }

pipeline {
    agent any

    environment {
        ENV_TYPE = "production"
        PORT = 3947
        NAMESPACE = "ironsamurai-site"
        REGISTRY_HOSTNAME = "maksimminakov42"
        REGISTRY = "registry.hub.docker.com"
        PROJECT = "iron-samurai"
        DEPLOYMENT_NAME = "iron-samurai-deployment"
        IMAGE_NAME = "${env.BUILD_ID}_${env.ENV_TYPE}_${env.GIT_COMMIT}"
        DOCKER_BUILD_NAME = "${env.REGISTRY_HOSTNAME}/${env.PROJECT}:${env.IMAGE_NAME}"
    }

    stages {
        stage('Clone repository') {
            steps {
                checkout scm
            }
        }

        stage('Unit tests') {
            steps {
                echo "Preparing and running unit tests..."
                script {
                    sh '''
                        export NVM_DIR="$HOME/.nvm"
                        [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                        nvm use --lts
                        npm install -g pnpm
                        pnpm install
                        pnpm test
                    '''
                }
            }
        }

        stage('Build Docker image') {
            steps {
                echo "Building Docker image..."
                script {
                    app = docker.build("${env.DOCKER_BUILD_NAME}")
                }
                echo "Docker image built successfully."
            }
        }

        stage('Push Docker image') {
            steps {
                echo "Pushing Docker image to registry..."
                script {
                    docker.withRegistry("https://${env.REGISTRY}", 'ironsamurai-site') {
                        app.push("${env.BUILD_ID}")
                    }
                }
                echo "Docker image pushed successfully."
            }
        }

        stage('Delete local Docker image') {
            steps {
                echo "Deleting local Docker image..."
                script {
                    sh "docker rmi -f ${env.DOCKER_BUILD_NAME}"
                }
            }
        }

        stage('Preparing deployment') {
            steps {
                echo "Preparing deployment..."
                script {
                    sh 'ls -ltr'
                    sh 'pwd'
                    sh "chmod +x preparingDeploy.sh"
                    sh "./preparingDeploy.sh ${env.REGISTRY_HOSTNAME} ${env.PROJECT} ${env.IMAGE_NAME} ${env.DEPLOYMENT_NAME} ${env.PORT} ${env.NAMESPACE}"
                    sh "cat deployment.yaml"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo "Deploying to Kubernetes..."
                withKubeConfig([credentialsId: 'prod-kubernetes']) {
                    sh 'kubectl apply -f deployment.yaml'
                    sh "kubectl rollout status deployment/${env.DEPLOYMENT_NAME} --namespace=${env.NAMESPACE}"
                    sh "kubectl get services -o wide"
                }
            }
        }
    }
}

