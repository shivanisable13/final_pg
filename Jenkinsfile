pipeline {

    agent any

    environment {

        IMAGE_NAME = "anuraj31/campusstay"

        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
    }

    stages {

        // =====================================================
        // CLONE SOURCE CODE
        // =====================================================

        stage('Clone Code') {

            steps {

                git branch: 'main',
                url: 'https://github.com/shivanisable13/final_pg.git'

                sh 'ls -la'
            }
        }

        // =====================================================
        // BUILD DOCKER IMAGE
        // =====================================================

        stage('Build Docker Image') {

            steps {

                sh '''
                docker build -t $IMAGE_NAME:latest .
                '''
            }
        }

        // =====================================================
        // PUSH IMAGE TO DOCKERHUB
        // =====================================================

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

        // =====================================================
        // DEPLOY MYSQL
        // =====================================================

        stage('Deploy MySQL') {

            steps {

                sh '''
                kubectl apply -f k8s/mysql.yaml
                '''
            }
        }

        // =====================================================
        // DEPLOY APPLICATION
        // =====================================================

        stage('Deploy Application') {

            steps {

                sh '''
                kubectl apply -f k8s/deployment.yaml

                kubectl apply -f k8s/service.yaml
                '''
            }
        }

        // =====================================================
        // ROLLING RESTART
        // =====================================================

        stage('Restart Deployment') {

            steps {

                sh '''
                kubectl rollout restart deployment campusstay-app
                '''
            }
        }

        // =====================================================
        // VERIFY
        // =====================================================

        stage('Verify Deployment') {

            steps {

                sh '''
                kubectl get pods

                kubectl get services
                '''
            }
        }
    }

    // =========================================================
    // POST BUILD
    // =========================================================

    post {

        success {

            echo '======================================='
            echo 'KUBERNETES DEPLOYMENT SUCCESSFUL'
            echo '======================================='

            echo 'Application URL:'
            echo 'http://54.91.235.69:30080'
        }

        failure {

            echo '======================================='
            echo 'DEPLOYMENT FAILED'
            echo '======================================='
        }
    }
}
