pipeline {
    environment {
        registry = "dvorkinguy/flask-dashboard"
        registryCredential = 'docker-hub-credentials' // Reference the credential ID
        dockerImage = ''
    }
    agent {
        label 'jenkins_worker' // Use the label of the desired node
    }
    stages {
        stage('checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/dvorkinguy/flask-dashboard.git'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

        stage('Test - Run Docker Container on Jenkins node') {
            steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
            }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
