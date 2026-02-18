pipeline {
    agent any
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                # Create a network so containers can talk to each other
                docker network create app-network || true
                
                # Clean up old containers
                docker rm -f backend1 backend2 || true
                
                # Run backends on the custom network
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                sleep 5
                '''
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb || true
                
                # Start NGINX on the same network (initially with default config)
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx
                
                sleep 5
                
                # Copy the config file FROM Jenkins workspace INTO the container
                # This fixes the "not a directory" mount error!
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                
                # Reload NGINX to apply the new configuration
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }
    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
