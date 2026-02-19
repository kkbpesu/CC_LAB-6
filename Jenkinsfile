pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                echo "Building backend Docker image..."

                docker rmi -f backend-app || true
                docker build -t backend-app ./backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                echo "Deploying backend containers..."

                docker network create app-network || true

                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                echo "Deploying nginx load balancer..."

                docker rm -f nginx-lb || true

                docker run -d --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf \
                  nginx

                docker restart nginx-lb
                '''
            }
        }

        stage('Verify Containers') {
            steps {
                sh '''
                echo "Running containers:"
                docker ps
                '''
            }
        }
    }

    post {
        success {
            echo " Pipeline executed successfully. NGINX load balancer is running."
        }

        failure {
            echo " Pipeline failed. Check console logs for errors."
        }
    }
}
