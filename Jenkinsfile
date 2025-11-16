pipeline {
    agent any
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nineslop/FlaskPyJenkins.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'pip install -r requirements.txt'
                sh 'pip install pytest'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh 'python -m pytest test_app.py -v'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    dockerImage = docker.build("flask-app:${env.BUILD_ID}")
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                sh '''
                docker stop flask-container || true
                docker rm flask-container || true
                docker run -d -p 5000:5000 --name flask-container flask-app:${BUILD_ID}
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                sh 'sleep 10'
                sh 'curl -f http://localhost:5000/health || exit 1'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh 'docker ps -a | grep flask'
        }
        success {
            echo 'Pipeline succeeded! Application is running on port 5000'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details'
        }
    }
}


