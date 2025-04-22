pipeline {
    agent {
        docker {
            image 'node:current-alpine3.21'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                    test -f ./build/index.html
                    npm test
                '''
            }
        }
    }
}