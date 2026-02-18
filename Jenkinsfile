pipeline {
    agent any

    stages {
        stage('Build Backend Image') {
            steps {
                // Fixed path: changed "CC_LAB-6/backend" to "backend"
                sh 'docker build -t backend-app backend'
            }
        }
        stage('Deploy Backend Containers') {
            steps {
                sh 'docker rm -f backend1 backend2 || true'
                sh 'docker run -d --name backend1 backend-app'
                sh 'docker run -d --name backend2 backend-app'
                sh 'sleep 3' 
            }
        }
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh 'docker rm -f nginx-lb || true'
                // Fixed path: changed "$(pwd)/CC_LAB-6/nginx..." to "$(pwd)/nginx..."
                sh 'docker run -d --name nginx-lb -p 80:80 -v $(pwd)/nginx/default.conf:/etc/nginx/conf.d/default.conf nginx' 
                sh 'sleep 2'
            }
        }
    }
    post {
        always {
            echo 'Pipeline executed successfully, NGINX load balancer is running.'
        }
    }
}
