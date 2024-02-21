@Library('shared-library') _
pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        APP_EXPOSED_PORT = "80"
        APP_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "${APP_NAME}-staging"
        PRODUCTION = "${APP_NAME}-prod"
        DOCKERHUB_ID = "sebastiensiddi"
        DOCKERHUB_PASSWORD = credentials('dockerhub_login')
        STG_API_ENDPOINT = "ip10-0-1-4-cnb20ibk3ds0ll6fnhag-1993.direct.docker.labs.eazytraining.fr"
        STG_APP_ENDPOINT = "ip10-0-1-4-cnb20ibk3ds0ll6fnhag-80.direct.docker.labs.eazytraining.fr"
        PROD_API_ENDPOINT = "ip10-0-1-5-cnb20ibk3ds0ll6fnhag-1993.direct.docker.labs.eazytraining.fr"
        PROD_APP_ENDPOINT = "ip10-0-1-5-cnb20ibk3ds0ll6fnhag-80.direct.docker.labs.eazytraining.fr"
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
                    sh 'docker build -t ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                sh '''
                    echo "Cleaning existing container if exist"
                    docker ps -a | grep -i $IMAGE_NAME && docker rm -f $IMAGE_NAME
                    docker run --name $IMAGE_NAME -d -p $APP_EXPOSED_PORT:$INTERNAL_PORT -e PORT=$INTERNAL_PORT ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                    sleep 5
                '''
                }
            }
        }
        stage('Test image') {
            agent any
            steps {
                script {
                    sh '''
                        curl http://192.168.1.76 | grep -q "Hello world!"
                    '''
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
                    docker push ${DOCKERHUB_ID}/$IMAGE_NAME:$IMAGE_TAG
                '''
                }
            }
        }
        stage('STAGING - Deploy app') {
            agent any
            steps {
                script {
                    sh """
                        echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
                        curl -v -X POST http://${STG_API_ENDPOINT}/staging -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    """
                }
            }
        }
        stage('PROD - Deploy app') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any

            steps {
                script {
                    sh """
                    echo  {\\"your_name\\":\\"${APP_NAME}\\",\\"container_image\\":\\"${CONTAINER_IMAGE}\\", \\"external_port\\":\\"${EXTERNAL_PORT}\\", \\"internal_port\\":\\"${INTERNAL_PORT}\\"}  > data.json
                    curl -v -X POST http://${PROD_API_ENDPOINT}/prod -H 'Content-Type: application/json'  --data-binary @data.json  2>&1 | grep 200
                    """
                }
            }
        }
    }
    post {
        always {
            script {
                slackNotifier currentBuild.result
            }
        }
    }
}