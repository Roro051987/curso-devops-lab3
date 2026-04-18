pipeline {
    agent {
        docker {
            image 'node:24'
            // args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    stages {
        stage('CI de la aplicación - dependencias') {
            steps {
                sh "npm install"
            }
        }
        stage('Pruebas unitarias - lint') {
            steps {
                sh "npm run lint"
            }
        }
        stage('Pruebas unitarias - test') {
            steps {
                sh "npm run test"
            }
        }
        stage('Pruebas unitarias - build') {
            steps {
                sh "npm run build"
            }
        }
        stage('Pruebas unitarias - build docker') {
            steps {
                sh "docker build -t curso-devops-lab3 ."
                
                }
            }
        }
    }    

}