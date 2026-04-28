pipeline {
    agent any

    stages {
        stage('Init') {
            steps {
                sh '''
                    docker stop $(docker ps -aq) 2>/dev/null || true
                    docker rm $(docker ps -aq) 2>/dev/null || true
                    docker network create app-network 2>/dev/null || true
                '''
            }
        }

        stage('Build') {
            steps {
                sh '''
                    docker build -t flask-app -f Dockerfile .
                    docker build -t nginx-proxy -f Dockerfile-nginx .
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker run -d --name flask-app --network app-network flask-app
                    docker run -d --name nginx-proxy --network app-network -p 80:5500 nginx-proxy
                '''
            }
        }
    }

    post {
        always {
            sh 'docker system prune -f 2>/dev/null || true'
        }
    }
}
