pipeline {
    agent any

    stages {
        stage('Build') {
            agent {
                docker {
                    label 'master'
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
        stage('Test') {
            node('master') {
                steps {
                sh '''
                    test -f build/index.html | if echo $? -eq "0"; then echo "The file exists"; else echo "The file cannot be found"; fi
                    CI=true npm test
                '''
                }
            }
        }
    }
}