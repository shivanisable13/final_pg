pipeline {

    agent any

    environment {

        IMAGE_NAME = "shivanisable/campusstay"

        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
    }

    stages {

        stage('Clone Code') {

            steps {

                git branch: 'main',
                url: 'https://github.com/shivanisable13/pg_web.git'
            }
        }

        stage('Build Docker Image') {

            steps {

                sh '''
                docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        stage('Push Docker Image') {

            steps {

                withCredentials([usernamePassword(
                    credentialsId: "$DOCKER_CREDENTIALS_ID",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {

                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin

                    docker push $IMAGE_NAME:latest
                    '''
                }
            }
        }

        stage('Deploy MySQL') {

            steps {

                sh '''
                kubectl apply -f k8s/mysql.yaml
                '''
            }
        }

        stage('Deploy Application') {

            steps {

                sh '''
                kubectl apply -f k8s/deployment.yaml

                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {

            steps {

                sh '''
                kubectl get pods

                kubectl get services
                '''
            }
        }
    }

    post {

        success {

            echo '======================================='
            echo 'KUBERNETES DEPLOYMENT SUCCESSFUL'
            echo '======================================='
        }

        failure {

            echo '======================================='
            echo 'DEPLOYMENT FAILED'
            echo '======================================='
        }
    }
}
