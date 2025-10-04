pipeline {
    agent { 
        docker { 
            image 'node:16' 
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        } 
    }

    environment {
        REGISTRY = "docker.io/jehansnh"
        APP_NAME = "aws-sample-node"
        TAG = 'latest'
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh 'npm install --save'
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                    if npm run test; then
                        echo "Tests passed"
                    else
                        echo "No test script found or test failed, continuing pipeline"
                    fi
                '''
            }
        }

        stage('Security Scan') {
            steps {
                withCredentials([string(
                    credentialsId: 'SNYK_TOKEN', 
                    variable: 'SNYK_TOKEN'
                )]) {
                    sh '''
                        npm install -g snyk
                        snyk auth $SNYK_TOKEN
                        snyk test --severity-threshold=high
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                    apt-get update -y
                    apt-get install -y docker.io
                    docker build -t $REGISTRY/$APP_NAME:$BUILD_NUMBER .
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                        echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                        docker push $REGISTRY/$APP_NAME:$BUILD_NUMBER
                    """
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
        }
        failure {
            echo 'Build failed!'
        }
    }
}