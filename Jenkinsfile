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
                    def buildTag = "latest" // Set the default tag as "latest"
                    if (env.BUILD_ID) {
                        buildTag = "build-${env.BUILD_ID}" // Use build number as the tag
                    }
                    img = "${registry}:${buildTag}"
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
                        // Also tag the image as 'latest' and push it
                        dockerImage.tag("${registry}:latest")
                        dockerImage.push()
                    }
                }
            }
        }
    }
}
