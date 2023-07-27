def img

pipeline {
    environment {
        registry = "dvorkinguy/flask-dashboard" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
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

        stage ('Stop previous running container'){
            steps{
                sh returnStatus: true, script: 'docker stop ${JOB_NAME}'
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force'
                sh returnStatus: true, script: 'docker rm ${JOB_NAME}'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    // Tag the image with the build number
                    img = registry + ":build-${env.BUILD_NUMBER}"
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
                        // Push the image tagged with the build number
                        dockerImage.push()

                        // Tag the image with 'latest' and push it
                        dockerImage.tag("${img}", "${registry}:latest")
                        dockerImage.push("${registry}:latest")
                    }
                }
            }
        }
    }
}
