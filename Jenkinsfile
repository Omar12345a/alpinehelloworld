pipeline {
    environment {
        IMAGE_NAME = 'alpinehelloworld'
        IMAGE_TAG = 'latest'
        STAGING = 'eazytraining-staging'
        PRODUCTION = 'eazytraining-production'
    }
    agent any
    stages {
        stage('Build image') {
            steps {
                script {
                    sh "docker build -t mechena/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }
        stage('Run container based on built image') {
            steps {
                script {
                    sh '''
                        docker run --name ${IMAGE_NAME} -d -p 80:5000 mechena/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5s
                    '''
                }
            }
        }
        stage('Test image') {
            steps {
                script {
                    sh 'curl http://localhost | grep -q "Hello world!"'
                }
            }
        }
        stage('Clean container') {
            steps {
                script {
                    sh '''
                        docker stop ${IMAGE_NAME}
                        docker rm ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage('Push image to staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'heroku_api_key', variable: 'HEROKU_API_KEY')]) {
                        sh '''
                            heroku container:login
                            heroku create ${STAGING} || echo "Project already exists"
                            heroku container:push -a ${STAGING} web
                            heroku container:release -a ${STAGING} web
                        '''
                    }
                }
            }
        }
        stage('Push image to production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            steps {
                script {
                    withCredentials([string(credentialsId: 'heroku_api_key', variable: 'HEROKU_API_KEY')]) {
                        sh '''
                            heroku container:login
                            heroku create ${PRODUCTION} || echo "Project already exists"
                            heroku container:push -a ${PRODUCTION} web
                            heroku container:release -a ${PRODUCTION} web
                        '''
                    }
                }
            }
        }
    }
}

