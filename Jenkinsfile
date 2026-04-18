pipeline {
    agent {
        docker {
            image 'node:24'
            // args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    stages {
        stage('CI de la aplicación') {
            steps {
                sh "npm install"
            }
        }

}