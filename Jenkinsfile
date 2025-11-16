pipeline {
    agent any
    
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nineslop/FlaskPyJenkins.git'
            }
        }
        
        stage('Setup Python Environment') {
            steps {
                sh 'python --version'
                sh 'sudo apt-get update && sudo apt-get install -y python3-venv || true'
                sh 'python -m venv venv || python -m ensurepip --default-pip'
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
                    sh "docker build -t flask-app:${env.BUILD_ID} ."
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    sh '''
                    docker stop flask-container || true
                    docker rm flask-container || true
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
        }
        failure {
            echo 'Pipeline failed! Check the logs for details'
        }
    }
}
