pipeline {
    agent any

    environment {
        REACT_APP_VERSION = "1.0.${BUILD_ID}"
        APP_NAME = 'learnjenkinsapp'
        AWS_DEFAULT_REGION = 'us-east-1'
        AWS_ECS_CLUSTER = 'LearnJenkinsApp-Cluster-Prod'
        AWS_ECS_SERVICE_PROD = 'LearnJenkinsApp-Service_Prod1'
        AWS_ECS_TD_PROD = 'LearnJenkinsApp-TaskDefinition-Prod1'
        AWS_DOCKER_REGISTRY = '238991998564.dkr.ecr.us-east-1.amazonaws.com'
        DOCKER_API_VERSION = '1.44'
    }

    stages {

        stage('Build') {
            agent {
                docker {
                    image 'node:current-alpine3.23'
                    reuseNode true
                }
            }
            steps {
                sh '''
                    node --version
                    npm --version
                    rm -rf node_modules
                    npm ci
                    npm audit fix --force
                    # npx update-browserslist-db@latest
                    npm run build
                '''
            }
        }
/*
        stage ('Build Docker Image') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "-u root -v /var/run/docker.sock:/var/run/docker.sock --entrypoint=''"
                }
            }
            steps {
                withCredentials([[ $class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh '''
                        # docker build -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        aws --version
                        aws ecr get-login-password | docker login --username AWS --password-stdin $AWS_DOCKER_REGISTRY
                        # docker build -t learnjenkinsapp .
                        docker buildx build --platform linux/amd64 -t $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION .
                        # docker tag learnjenkinsapp:latest 238991998564.dkr.ecr.us-east-1.amazonaws.com/learnjenkinsapp:latest
                        # docker push 238991998564.dkr.ecr.us-east-1.amazonaws.com/learnjenkinsapp:latest
                        docker push $AWS_DOCKER_REGISTRY/$APP_NAME:$REACT_APP_VERSION
                    '''
                }
            }
        }

        stage ('Deploy to AWS') {
            agent {
                docker {
                    image 'my-aws-cli'
                    reuseNode true
                    args "--entrypoint=''"
                }
            }
            steps {
                withCredentials([[ $class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh '''
                        # aws --version
                        sed -i "s/#APP_VERSION#/$REACT_APP_VERSION/g" aws/task-definition-prod.json
                        cat aws/task-definition-prod.json
                        LATEST_TD_REVISION=$(aws ecs register-task-definition --cli-input-json file://aws/task-definition-prod.json | jq '.taskDefinition.revision')
                        aws ecs update-service --cluster $AWS_ECS_CLUSTER --service $AWS_ECS_SERVICE_PROD --task-definition $AWS_ECS_TD_PROD:$LATEST_TD_REVISION
                        aws ecs wait services-stable --cluster $AWS_ECS_CLUSTER --services $AWS_ECS_SERVICE_PROD
                    '''
                }
            }
        }
    */
    }   
 }