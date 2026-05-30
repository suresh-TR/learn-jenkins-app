pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '921b1f53-85ad-4f95-afce-ba8532633848'
        NETLIFY_AUTH_TOKEN = credentials('Netlify_token')
        REACT_APP_VERSION = "1.0.$BUILD_ID"
    }
    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                }
            }
            steps {
                sh '''
                    aws --version
                '''
            }
        }
        stage('Build') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        stage ('Test') {
            parallel {
                stage('Unit Tests') {
                    agent {
                        docker {
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            test -f build/index.html
                            npm run test
                            echo 'Test stage'
                        '''
                    }
                }

                stage('E2E') {
                    agent {
                        docker {
                            image 'my-playwright'
                            reuseNode true
                        }
                    }
                    steps {
                        sh '''
                            serve -s build &
                            sleep 5
                            npx playwright test --reporter=html
                        '''
                    }
                }
            }
        }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to staging SITE ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json deploy-output.json
                    node-jq -r '.deploy_url' deploy-output.json
                '''
            }
        }
        
        stage('Deploy Production') {
            agent {
                docker {
                    image 'my-playwright'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    netlify --version
                    echo "Deploy to production SITE ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --auth $NETLIFY_AUTH_TOKEN --dir=build --prod

                '''
            }
        }
        
    }
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
