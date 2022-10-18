pipeline {
    environment {
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "jenkinstraining-staging"
        PRODUCTION = "jenkinstraining-prod"
    }
    agent none
    stages {
        stage('Build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t  matthewmurdauck/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        } 
        stage('Run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
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
                        curl http://192.168.1.24 | grep -q "Hello world!"
                    '''
                }
            }
        }
        stage('Clean container') {
            agent any
            steps {
                script {
                    sh '''
                       docker stop ${IMAGE_NAME}
                       docker rm  ${IMAGE_NAME}
                    '''
                }
            }
        }
        stage('Push image in staging and deploy it') {
         when {
                 expression { GIT_BRANCH == 'origin/master'}
              }
         agent any
         environment {
            HEROKU_API_KEY = credentials('heroku_api_key')
         }
         steps {
            script{
                sh '''
                heroku container:login
                heroku create $STAGING || echo "project already exist"
                heroku container:push -a $STAGING web
                '''
            }
         }
        }

        stage('Push image in production and deploy it') {
         when {
                 expression { GIT_BRANCH == 'origin/master'}
              }
         agent any
         environment {
            HEROKU_API_KEY = credentials('heroku_api_key')
         }
         steps {
            script{
                sh '''
                heroku container:login
                heroku create $PRODUCTION || echo "project already exist"
                heroku container:push -a $PRODUCTION web
                '''
            }
         }
        }
    }
}
