// def tagAndPush(String localImage, String repo, String registry, String credential) {

//     docker.withRegistry(registry, credential) {
//         sh "docker tag ${localImage} ${repo}:latest"
//         sh "docker tag ${localImage} ${repo}:${env.BUILD_NUMBER}"
//         sh "docker tag ${localImage} ${repo}:${env.APP_SEMANTIC_VERSION}"
//         sh "docker push ${repo}:latest"
//         sh "docker push ${repo}:${env.BUILD_NUMBER}"
//         sh "docker push ${repo}:${env.APP_SEMANTIC_VERSION}"
//     }

// }


pipeline {
    agent any
    stages {
        stage("Integracion continua") {
            agent {
                docker {
                    image "node:24"
                    reuseNode true
                }
            }
            stages {
                stage("CI de la aplicacion - version") {
                    steps {
                        script {
                            env.APP_SEMANTIC_VERSION = sh(
                                script: 'npm pkg get version | tr -d \'"\'',
                                returnStdout: true
                            ).trim()
                            echo "Version semantica detectada: ${env.APP_SEMANTIC_VERSION}"
                        }
                    }
                }
                stage("CI de la aplicacion - dependencias") {
                    steps {
                        sh "npm install"
                    }
                }
                stage("CI de la aplicacion - lint") {
                    steps {
                        sh "npm run lint"
                    }
                }
                stage("CI de la aplicacion - testcov") {
                    steps {
                        sh "npm run test:cov"
                    }
                }
                stage("CI de la aplicacion - build") {
                    steps {
                        sh "npm run build"
                    }
                }
            }
        }
        stage('Quality Assurance - Analisis Sonar') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli'
                    args '--network=devops-infra_default'
                    reuseNode true
                }
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                    echo "Esperando SonarQube listo..."

                    until curl -s http://sonarqube:9000/api/system/status | grep -q '"status":"UP"'; do
                        echo "SonarQube no está listo aún..."
                        sleep 5
                    done

                    echo "SonarQube listo, ejecutando análisis..."
                    
                    sonar-scanner \
                        -Dsonar.projectKey=curso-devops-lab3 \
                        -Dsonar.projectName=curso-devops-lab3 \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=$SONAR_HOST_URL \
                        -Dsonar.token=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }
        stage('CD de la aplicacion - build dockerfile') {
            
            steps {
                echo "APP_SEMANTIC_VERSION ${env.APP_SEMANTIC_VERSION}"
                echo "BUILD_NUMBER ${env.BUILD_NUMBER}"
                script {
                    sh 'docker version'
                    sh 'docker build -t curso-devops-lab3 .'

                    docker.withRegistry('https://index.docker.io/v1/', 'credencial_dh') {
                        sh "docker tag curso-devops-lab3 roro05/curso-devops-lab3:latest"
                        sh "docker tag curso-devops-lab3 roro05/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                        sh "docker tag curso-devops-lab3 roro05/curso-devops-lab3:${env.BUILD_NUMBER}"
                        
                        sh "docker push roro05/curso-devops-lab3:latest" 
                        sh "docker push roro05/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                        sh "docker push roro05/curso-devops-lab3:${env.BUILD_NUMBER}"
         
                    }

                    docker.withRegistry("https://ghcr.io", "credencial_gh") {
                        sh "docker tag curso-devops-lab3 ghcr.io/roro051987/curso-devops-lab3:latest"
                        sh "docker tag curso-devops-lab3 ghcr.io/roro051987/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                        sh "docker tag curso-devops-lab3 ghcr.io/roro051987/curso-devops-lab3:${env.BUILD_NUMBER}"
                        
                        sh "docker push ghcr.io/roro051987/curso-devops-lab3:latest"
                        sh "docker push ghcr.io/roro051987/curso-devops-lab3:${env.APP_SEMANTIC_VERSION}"
                        sh "docker push ghcr.io/roro051987/curso-devops-lab3:${env.BUILD_NUMBER}"
                    } 
                }
            } 
        }
        // stage("CD - Despliegue continuo en develop"){
        //     agent {
        //         docker {
        //             image 'alpine/k8s:1.34.6'
        //             reuseNode true
        //         }
        //     }
        //     steps{
        //         script {
        //             if (!env.APP_SEMANTIC_VERSION?.trim()) {
        //                 error("APP_SEMANTIC_VERSION no definida para el despliegue")
        //             }
        //         }
        //         withKubeConfig([credentialsId: 'credencial-k8']) {
        //             sh """
        //                 kubectl -n ${env.K8S_NAMESPACE} set image deployment/${env.K8S_DEPLOYMENT} ${env.K8S_CONTAINER}=${env.DH_REPO}:${env.APP_SEMANTIC_VERSION}
        //                 kubectl -n ${env.K8S_NAMESPACE} rollout status deployment/${env.K8S_DEPLOYMENT}
        //             """
        //         }
        //     }
        // }
    }
}