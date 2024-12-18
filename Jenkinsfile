pipeline {
    agent any

    stage('Install dependencies') {
        steps {
            sh 'npm install'
        }
    }

    stage('Test') {
        steps {
            sh 'echo "testing applications ..."'
        }
    }

    stage('Deploy nodejs application') {
        steps {
            sh 'echo "deploying applications..."'
        }
    }
}

