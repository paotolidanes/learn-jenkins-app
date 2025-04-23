pipeline {
    agent any
    
    environment {
        NETLIFY_SITE_ID = '769a07e2-c4d8-4bd6-ae4e-b25802502d97'
    }
    
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:current-alpine3.21'
                    reuseNode true
                    args '-u root:root'
                }
            }
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

        stage('Run Tests') {
            parallel {

                stage('Unit Test') {
                    agent {
                        docker {
                            image 'node:current-alpine3.21'
                            reuseNode true
                            args '-u root:root'
                        }
                    }

                    steps {
                        sh '''
                            npm test
                        '''
                    }

                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E Test') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                            reuseNode true
                            args '-u root:root'
                        }
                    }

                    steps {
                        sh '''
                            npm install serve
                            ./node_modules/.bin/serve -s build &
                            sleep 10s
                            npx playwright test --reporter=html
                        '''
                    }

                    post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            agent {
                docker {
                    image 'node:current-alpine3.21'
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                sh '''
                    npm install netlify-cli
                    ./node_modules/.bin/netlify --version
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                '''
            }
        }
    }

}