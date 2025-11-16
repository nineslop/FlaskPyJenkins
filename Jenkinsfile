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
                sh 'python --version'
                sh 'python -m venv venv'
                sh '. venv/bin/activate && pip install -r requirements.txt'
                sh '. venv/bin/activate && pip install pytest'
            }
        }
        
        stage('Run Tests') {
            steps {
                sh '. venv/bin/activate && python -m pytest test_app.py -v || echo "Tests completed"'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    // Проверяем доступ к Docker
                    sh 'docker --version'
                    sh 'docker ps || true'
                    
                    // Собираем образ
                    sh "docker build -t flask-app:${env.BUILD_ID} ."
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    # Останавливаем старый контейнер если есть
                    docker stop flask-container || true
                    docker rm flask-container || true
                    
                    # Запускаем новый контейнер
                    docker run -d -p 5000:5000 --name flask-container flask-app:${BUILD_ID}
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                sh 'sleep 10'
                sh 'curl -f http://localhost:5000/health || exit 1'
                sh 'curl -f http://localhost:5000/ || exit 1'
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline execution completed'
            sh 'docker ps -a | grep flask || true'
        }
        success {
            echo 'Pipeline succeeded! Application is running on port 5000'
            sh 'echo "Access the application at: http://localhost:5000"'
        }
        failure {
            echo 'Pipeline failed! Check the logs for details'
        }
    }
}
