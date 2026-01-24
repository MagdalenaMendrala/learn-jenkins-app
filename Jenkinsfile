pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = 'f33ff802-69f8-489d-b112-60b86fa537cf'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    echo "small change"
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Test') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            echo 'Test stage'
                            test -f build/index.html
                            npm test
                        '''
                    }
                    post {
                        always {
                            junit 'jest-results/junit.xml'
                        }
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            npm install serve
                            node_modules/.bin/serve -s build &
                            sleep 10
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

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                  npm install netlify-cli@20.1.1
                  ./node_modules/.bin/netlify --version
                  echo "Deploying to staging site: $NETLIFY_SITE_ID"
                  ./node_modules/.bin/netlify status
                  ./node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                  npm install netlify-cli@20.1.1
                  ./node_modules/.bin/netlify --version
                  echo "Deploying to production site: $NETLIFY_SITE_ID"
                  ./node_modules/.bin/netlify status
                  ./node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
                agent {
                    docker {
                        image 'mcr.microsoft.com/playwright:v1.57.0-noble'
                        reuseNode true
                    }
                }
                environment {
                    CI_ENVIRONMENT_URL = 'https://steady-bubblegum-067976.netlify.app'
                }
                steps {
                    sh '''
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
