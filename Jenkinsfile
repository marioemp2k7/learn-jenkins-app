pipeline {
    agent any

    environment {
        NETLIFY_SITE_ID = '22917062-6731-42da-8185-4acdd90ccc7f'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
    }

    stages {

        stage ('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                }
            }
            steps {
                sh '''
                    aws --version
                    aws s3 ls
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
                            image 'node:18-alpine'
                            reuseNode true
                        }
                    }

                    steps {
                        sh '''
                            test -f build/index.html | if echo $? -eq "0"; then echo "The file exists"; else echo "The file cannot be found"; fi
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
                                image 'my-playright'
                                reuseNode true
                            }
                        }
                        steps {
                                sh '''
                                serve -s build &
                                sleep 10
                                npx playwright test --reporter=html
                                '''
                        }
                        post {
                        always {
                            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: '/var/jenkins_home/workspace/learn-jenkins/playwright-report', reportFiles: 'index.html', reportName: 'Playwright Local', reportTitles: '', useWrapperFileDirectly: true])
                        }
                    }
                }
            }        
        } 
        

        stage('Deploy staging') {
            agent {
                docker {
                    image 'my-playright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'STAGING_URL_TO_BE_SET'
            }

            steps {
                sh '''
                    netlify --version
                    echo "Deploying to staging. Site ID: $NETLIFY_SITE_ID"
                    netlify status
                    netlify deploy --dir=build --json > /var/jenkins_home/workspace/learn-jenkins/deploy_output.json
                    CI_ENVIRONMENT_URL=$(jq -r '.deploy_url' /var/jenkins_home/workspace/learn-jenkins/deploy_output.json)
                    echo "Deployed to URL: $CI_ENVIRONMENT_URL"
                    npx playwright test  --reporter=html
                '''
            }

            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/jenkins_home/workspace/learn-jenkins/playwright-report', reportFiles: 'index.html', reportName: 'Staging E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }

        stage('Deploy Prod') {
            agent {
                docker {
                    image 'my-playright'
                    reuseNode true
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://staging--mariusbreta-learn-gitlab.netlify.app'
            }

            steps {
                    sh '''
                    netlify --version
                    echo "Deploying to Site ID: ${NETLIFY_SITE_ID}"
                    netlify status      
                    netlify deploy --dir=build --alias=staging
                    npx playwright test --reporter=html
                    '''
            }
            post {
                always {
                    publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, icon: '', keepAll: false, reportDir: '/var/jenkins_home/workspace/learn-jenkins/playwright-report', reportFiles: 'index.html', reportName: 'Playwright E2E', reportTitles: '', useWrapperFileDirectly: true])
                }
            }
        }
    }   
 }