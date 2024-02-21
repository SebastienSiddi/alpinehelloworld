pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        APP_EXPOSED_PORT = "80"
        APP_NAME = "app_alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "${APP_NAME}-staging"
        PRODUCTION = "${APP_NAME}-production"
        DOCKERHUB_ID = "sebastiensiddi"
        DOCKERHUB_PASSWORD = credentials('dockerhub_login')
        STAGING_API_ENDPOINT = "http://192.168.1.76:1993"
        STAGING_APP_ENDPOINT = "http://192.168.1.76:80"
        PRODUCTION_API_ENDPOINT = "http://192.168.1.76:1994"
        PRODUCTION_APP_ENDPOINT = "http://192.168.1.76:80"
        INTERNAL_PORT = "5000"
        EXTERNAL_PORT = "${APP_EXPOSED_PORT}"
        CONTAINER_IMAGE = "${DOCKERHUB_ID}/${IMAGE_NAME}:${IMAGE_TAG}"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t $CONTAINER_IMAGE .'
                }
            }
        }
        stage('Run container based on build image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run -d -p 80:5000 -e PORT=5000 --name $IMAGE_NAME $CONTAINER_IMAGE
                        sleep 5
                    '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh 'curl http://192.168.1.76 | grep -q "Hello world!"'
                }
            }
        }
        stage('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                        docker stop $IMAGE_NAME
                        docker rm $IMAGE_NAME
                    '''
                }
            }
        }
        stage('Login and Push Image on docker hub') {
            agent any
            steps {
                script {
                    sh '''
                        echo $DOCKERHUB_PASSWORD_PSW | docker login -u $DOCKERHUB_PASSWORD_USR --password-stdin
                        docker push $DOCKERHUB_ID/$IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }
        stage('STAGING - Deploy app') {
            agent any
            steps {
                script {
                    sh '''
                        echo  {\\"Sébastien Siddi\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}00\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
                        curl -v -X POST $STAGING_API_ENDPOINT -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    '''
                }
            }
        }
        stage('PROD - Deploy app') {
            when {
                expression { GIT_BRANCH == 'origin/main' }
            }
            agent any
            steps {
                script {
                    sh '''
                    echo  {\\"Sébastien Siddi\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
                    curl -v -X POST $PRODUCTION_API_ENDPOINT -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    '''
                }
            }
        }
    }
}
