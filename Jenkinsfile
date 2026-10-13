
pipeline {
    agent any

    environment {
        APP_NAME = 'toqa-python-app'
        REPO_URL = 'https://github.com/toqaosama/jenkinsnode.js.git'
        DOCKER_CREDENTIALS_ID = 'docker'
    }

    stages {
        stage('Getting Repo Files') {
            steps {
                git branch: 'main',
                    credentialsId: '68',
                    url: 'https://github.com/toqaosama/jenkinsnode.js.git'

                sh 'ls -la'
            }
        }

        stage('Check Python App') {
            steps {
                sh '''
                    test -f app.py
                    test -f Dockerfile
                    echo "Python app and Dockerfile found"
                    cat app.py
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      -t ${APP_NAME}:${BUILD_NUMBER} \
                      -t ${APP_NAME}:latest .
                '''
            }
        }

        stage('Test Python App') {
            steps {
                sh '''
                    docker run --rm ${APP_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x
                        echo "$DOCKER_PASSWORD" | docker login \
                          --username "$DOCKER_USERNAME" \
                          --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
           usernamePassword(
                        credentialsId: "${DOCKER_CREDENTIALS_ID}",
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        docker tag ${APP_NAME}:${BUILD_NUMBER} \
                          "$DOCKER_USERNAME/${APP_NAME}:${BUILD_NUMBER}"

                        docker tag ${APP_NAME}:${BUILD_NUMBER} \
                          "$DOCKER_USERNAME/${APP_NAME}:latest"

                        docker push "$DOCKER_USERNAME/${APP_NAME}:${BUILD_NUMBER}"
                        docker push "$DOCKER_USERNAME/${APP_NAME}:latest"
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                sh '''
                    docker rm -f ${APP_NAME}-${BUILD_NUMBER} || true

                    docker run --name ${APP_NAME}-${BUILD_NUMBER} \
                      ${APP_NAME}:${BUILD_NUMBER}
                '''
            }
        }
    }

    post {
        success {
            echo 'SUCCESS: Python app built, tested, and pushed to Docker Hub!'
        }

        failure {
            echo 'FAILED: Check the failed stage in Jenkins Console Output.'
        }

        always {
            sh 'docker logout || true'
        }
    }
}
