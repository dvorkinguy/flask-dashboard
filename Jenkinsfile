def img

pipeline {
    environment {
        registry = "dvorkinguy/flask-dashboard" // To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'docker-hub-credentials'
        dockerImage = ''
    }
    agent {
        label 'jenkins_worker'
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
                    img = registry + ":build-${env.BUILD_NUMBER}" // Use the build number as the tag
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

        stage('Test - Run Docker Container on Jenkins node') {
            steps {
                sh label: '', script: "docker stop flask-dashboard-ci || true"
                sh label: '', script: "docker rm flask-dashboard-ci || true"
                sh label: '', script: "docker run -d --name flask-dashboard-ci -p 5000:5000 ${img}"
            }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        dockerImage.push()

                        // Tag and push the image as 'latest'
                        dockerImage.tag("${img}", "${registry}:latest")
                        dockerImage.push("${registry}:latest")
                    }
                }
            }
        }
    }
}
