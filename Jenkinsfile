pipeline {
    agent any

    environment {
        // Docker environment (so Jenkins uses your local Docker)
        DOCKER_HOST = 'unix:///Users/aathreya/.docker/run/docker.sock'
        DOCKER_CERT_PATH = '/Users/aathreya/.docker'
        DOCKER_TLS_VERIFY = '1'

        // DockerHub credentials (already created in Jenkins)
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo '📦 Cloning GitHub repository...'
                git branch: 'main', url: 'https://github.com/aathreya-sharma/flask-mysql-app.git'
            }
        }

        stage('Build') {
            steps {
                echo '🔧 Building Docker images...'
                retry(3) {
                    sh '''
                        echo "⏳ Running docker-compose build..."
                        docker-compose build || { 
                            echo "⚠️ Build failed, retrying in 5s..."; 
                            sleep 5; 
                            false; 
                        }
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                echo '🧪 Running containers and tests...'
                sh '''
                    docker-compose up -d
                    echo "⏳ Waiting for services to start..."
                    sleep 15
                    curl --fail http://localhost:5001/ || (echo "❌ App test failed!" && exit 1)
                '''
            }
        }

        stage('Push to Docker Hub') {
            steps {
                echo '📤 Pushing Docker image to Docker Hub...'
                script {
                    sh '''
                        echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                        docker build -t $DOCKERHUB_CREDENTIALS_USR/flask-mysql-app:latest .
                        docker push $DOCKERHUB_CREDENTIALS_USR/flask-mysql-app:latest
                    '''
                }
            }
        }

        stage('Cleanup') {
            steps {
                echo '🧹 Cleaning up containers and images...'
                sh '''
                    docker-compose down -v || true
                    docker rmi $DOCKERHUB_CREDENTIALS_USR/flask-mysql-app:latest || true
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Build, test, and push completed successfully!'
        }
        failure {
            echo '❌ Build Failed — Check Logs!'
        }
    }
}