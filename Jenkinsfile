pipeline {
    agent {
        kubernetes {
            label 'ci-cd-pipeline'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: jenkins-agent
spec:
  containers:
  - name: docker
    image: docker:20.10.8
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-socket
      mountPath: /var/run/docker.sock
  - name: nodejs
    image: node:16
    command:
    - cat
    tty: true
  volumes:
  - name: docker-socket
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }
    environment {
        DOCKER_IMAGE_FRONTEND = "your-dockerhub-username/frontend-app"
        DOCKER_IMAGE_BACKEND = "your-dockerhub-username/backend-app"
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo.git'
            }
        }
        stage('Build Frontend') {
            steps {
                container('nodejs') {
                    sh """
                    cd frontend
                    npm install
                    npm run build
                    """
                }
            }
        }
        stage('Build and Push Docker Images') {
            steps {
                container('docker') {
                    sh """
                    docker build -t $DOCKER_IMAGE_FRONTEND ./frontend
                    docker build -t $DOCKER_IMAGE_BACKEND ./backend
                    docker login -u your-dockerhub-username -p your-dockerhub-password
                    docker push $DOCKER_IMAGE_FRONTEND
                    docker push $DOCKER_IMAGE_BACKEND
                    """
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('docker') {
                    sh """
                    kubectl apply -f k8s/deployment.yaml -n my-app
                    kubectl apply -f k8s/service.yaml -n my-app
                    """
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline execution completed!"
        }
    }
}
