pipeline {
    environment {
        IMAGE_NAME = "alplinehelloworld"
        IMAGE_TAG = "latest"
        DOCKERHUB_ID = "sebastiensiddi"
        DOCKERHUB_CREDS = credentials('dockerhub_login')
        STAGING = "staging"
        PRODUCTION = "production"
        APP_NAME = "alpinehelloworld"
        STAGING_API_ENDPOINT = "http://192.168.1.76:1993"
        STAGING_APP_ENDPOINT = "http://192.168.1.76:83"
        PRODUCTION_API_ENDPOINT = "http://192.168.1.76:1994"
        PRODUCTION_APP_ENDPOINT = "http://192.168.1.76:84"
        INTERNAL_PORT = "5000"
        STAGING_EXTERNAL_PORT = "83"
        PRODUCTION_EXTERNAL_PORT = "84"
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
                        docker login -u="$DOCKERHUB_CREDS_USR" -p="$DOCKERHUB_CREDS_PSW"
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
                        echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"$STAGING_EXTERNAL_PORT\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
                        curl -v -X POST ${STAGING_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
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
                    echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"$PRODUCTION_EXTERNAL_PORT\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
                    curl -v -X POST ${PRODUCTION_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    '''
                }
            }
        }
    }
}