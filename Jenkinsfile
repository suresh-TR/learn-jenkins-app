pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.$BUILD_ID"
        AWS_S3_JENKINS_BUCKET = credentials('aws-s3-bucket')
        AWS_DEFAULT_REGION = 'us-east-1'
    }
    stages {
        stage('Deploy to AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    reuseNode true
                    args "-u root --entrypoint=''"
                }
            }
            // environment {
            //     AWS_S3_BUCKET = 'learn-jenkins-20240530307'
            // }
            steps {
                withCredentials([usernamePassword(credentialsId: 'my-aws-credentials', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
                    sh '''
                        aws --version
                        yum install jq -y
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq -r '.taskDefinition.revision')
                        echo $LATEST_TD_REVISION
                        aws ecs update-service --cluster LearnJenkinsApp-Cluster-Prod-test --service LearnJenkinsApp-TaskDefinition-Prod-test-service --task-definition LearnJenkinsApp-TaskDefinition-Prod-test:$LATEST_TD_REVISION
                    '''
                }
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
        //                     image 'my-playwright'
        //                     reuseNode true
        //                 }
        //             }
        //             steps {
        //                 sh '''
        //                     serve -s build &
        //                     sleep 5
        //                     npx playwright test --reporter=html
        //                 '''
        //             }
        //         }
        //     }
        // }
        // stage('Deploy Staging') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             netlify --version
        //             echo "Deploy to staging SITE ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --dir=build --json deploy-output.json
        //             jq -r '.deploy_url' deploy-output.json
        //         '''
        //     }
        // }
        
        // stage('Deploy Production') {
        //     agent {
        //         docker {
        //             image 'my-playwright'
        //             reuseNode true
        //         }
        //     }
        //     steps {
        //         sh '''
        //             netlify --version
        //             echo "Deploy to production SITE ID: $NETLIFY_SITE_ID"
        //             netlify status
        //             netlify deploy --auth $NETLIFY_AUTH_TOKEN --dir=build --prod

        //         '''
        //     }
        // }
        
    }
    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Playwright HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
}
