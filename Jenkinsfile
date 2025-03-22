pipeline {
    agent any
    environment {
        BACKEND_IMAGE = "jeeva3104/backend-app:latest"
        FRONTEND_IMAGE = "jeeva3104/frontend-app:latest"
        BACKEND_CONTAINER = "backend-container"
        FRONTEND_CONTAINER = "frontend-container"
        REGISTRY_CREDENTIALS = "docker-jeeva"  // Jenkins credentials ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-jeeva', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/Jeeva-21BSR017/devopsk8s.git", branch: 'main'
                }
            }
        }

        stage('Build Backend Image') {
            steps {
                sh 'docker build -t $BACKEND_IMAGE ./backend'
            }
        }

        stage('Build Frontend Image') {
            steps {
                sh 'docker build -t $FRONTEND_IMAGE ./frontend'
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-jeeva', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Backend Image') {
            steps {
                sh 'docker push $BACKEND_IMAGE'
            }
        }

        stage('Push Frontend Image') {
            steps {
                sh 'docker push $FRONTEND_IMAGE'
            }
        }

        stage('Stop & Remove Existing Backend Container') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$BACKEND_CONTAINER)" ]; then
                        docker stop $BACKEND_CONTAINER || true
                        docker rm $BACKEND_CONTAINER || true
                    fi
                    '''
                }
            }
        }

        stage('Stop & Remove Existing Frontend Container') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$FRONTEND_CONTAINER)" ]; then
                        docker stop $FRONTEND_CONTAINER || true
                        docker rm $FRONTEND_CONTAINER || true
                    fi
                    '''
                }
            }
        }

        stage('Run Backend Container') {
            steps {
                sh 'docker run -d -p 5001:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE'
            }
        }

        stage('Run Frontend Container') {
            steps {
                sh 'docker run -d -p 3000:3000 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE'
            }
        }
    }

    post {
        success {
            echo "Backend and frontend deployed successfully!"
        }
        failure {
            echo "Something went wrong."
        }
    }
}
