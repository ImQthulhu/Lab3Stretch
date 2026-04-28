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

        stage('Trivy Scan') {
            steps {
                sh 'trivy fs --output trivy-report.txt .'
            }
            post {
                always {
                    archiveArtifacts artifacts: 'trivy-report.txt'
                }
            }
        }

        stage('Approve') {
            steps {
                input message: 'Trivy scan complete. Approve deployment?', ok: 'Deploy'
            }
        }

        stage('Unit Tests') {
            steps {
                script {
                    sh 'pip3 install pytest flask --break-system-packages'
                    def result = sh(script: 'python3 -m pytest test_app.py -v', returnStatus: true)
                    if (result != 0) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker run -d --name flask-app --network app-network flask-app
                    docker run -d --name nginx-proxy --network app-network -p 8082:80 nginx-proxy
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
