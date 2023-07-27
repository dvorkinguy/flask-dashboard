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
                    img = registry + ":${env.BUILD_NUMBER}" // Use the build number as the tag
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

                        // Tag the image with the build number and push it
                        def buildTag = "build-${env.BUILD_NUMBER}"
                        dockerImage.tag("${img}", "${registry}:${buildTag}")
                        dockerImage.push("${registry}:${buildTag}")

                        // Update the 'latest' tag to point to the new image
                        dockerImage.tag("${img}", "${registry}:latest")
                        dockerImage.push("${registry}:latest")
                    }
                }
            }
        }
    }
}
