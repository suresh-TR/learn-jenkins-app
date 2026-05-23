pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '921b1f53-85ad-4f95-afce-ba8532633848'
        NETLIFY_AUTH_TOKEN = credentials('Netlify_token')
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
                    ls -la
                    node --version
                    npm --version
                    npm ci
                    npm run build
                    ls -la
                '''
            }
        }
        // stage ('Test') {
        //     parallel {
        //         stage('Unit Tests') {
        //             agent {
        //                 docker {
        //                     image 'node:18-alpine'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     test -f build/index.html
        //                     npm run test
        //                     echo 'Test stage'
        //                 '''
        //             }
        //         }

        //         stage('E2E') {
        //             agent {
        //                 docker {
        //                     image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     npm install serve
        //                     node_modules/.bin/serve -s build &
        //                     sleep 5
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //         }
        //     }
        // }
        stage('Deploy Staging') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1 node-jq
                    node_modules/.bin/netlify --version
                    echo "Deploy to staging SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --dir=build --json > deploy-output.json
                    node_modules/.bin/jq -r '.deploy_url' > deploy-output.json
                '''
            }
        }
        stage('Approval') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: 'Ready to deploy?', ok: 'Yes, I am sure to deploy'
                }
            }
        }
        stage('Deploy Production') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify --version
                    echo "Deploy to production SITE ID: $NETLIFY_SITE_ID"
                    node_modules/.bin/netlify status
                    node_modules/.bin/netlify deploy --auth $NETLIFY_AUTH_TOKEN --dir=build --prod

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
