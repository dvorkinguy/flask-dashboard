pipeline {
    agent any

    stages {
        stage('Build Image'){
            steps{
                sh "docker build -t flask-dashboard:${params.flask_version} ."      
            }
        }
    }
}
