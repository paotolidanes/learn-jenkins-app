pipeline {
    agent any
    
    environment {
        NETLIFY_SITE_ID = '769a07e2-c4d8-4bd6-ae4e-b25802502d97'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
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
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E Local Report', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }
        }

        stage('Approval to Prod') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input cancel: 'Denied', message: 'Do you wish to deploy to production?', ok: 'Yes, I am sure!'
                }
            }
        }

        stage('Deploy to Staging') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    args '-u root:root'
                }
            }

            environment {
                CI_ENVIRONMENT_URL = ""
            }

            steps {
                sh '''
                    npm install netlify-cli node-jq
                    ./node_modules/.bin/netlify --version
                    ./node_modules/.bin/netlify status
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    ./node_modules/.bin/netlify deploy --dir=build --json > deploy-staging-output.json
                    CI_ENVIRONMENT_URL=$(./node_modules/.bin/node-jq -r '.deploy_url' deploy-staging-output.json)
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Staging E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy to Prod') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                    args '-u root:root'
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://frolicking-blancmange-8add68.netlify.app'
            }

            steps {
                sh '''
                    node --version
                    npm install netlify-cli
                    ./node_modules/.bin/netlify --version
                    ./node_modules/.bin/netlify status
                    echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
                    ./node_modules/.bin/netlify deploy --dir=build --prod
                    npx playwright test --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright Prod E2E Report', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }
}