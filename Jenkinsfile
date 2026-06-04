pipeline {

    agent any

    tools {
        maven 'maven'
    }

    environment {
        IMAGE_NAME = "java-app"
        EC2_HOST = credentials('ec2-host')
    }

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                git(
                    branch: 'main',
                    url: 'https://github.com/vishnuvardhandhadam/jenkins-java-project1.git'
                )
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh """
                    echo \$DOCKER_PASS | docker login \
                    -u \$DOCKER_USER \
                    --password-stdin

                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    \$DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}

                    docker tag ${IMAGE_NAME}:${BUILD_NUMBER} \
                    \$DOCKER_USER/${IMAGE_NAME}:latest

                    docker push \$DOCKER_USER/${IMAGE_NAME}:${BUILD_NUMBER}

                    docker push \$DOCKER_USER/${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('Deploy Application') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sshagent(['ec2-ssh-key']) {

                        sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@\$EC2_HOST "

                        echo \$DOCKER_PASS | docker login \
                        -u \$DOCKER_USER \
                        --password-stdin

                        docker pull \$DOCKER_USER/${IMAGE_NAME}:latest

                        docker stop java-app || true

                        docker rm java-app || true

                        docker run -d \
                        --name java-app \
                        --restart always \
                        -p 8180:8080 \
                        \$DOCKER_USER/${IMAGE_NAME}:latest
                        "
                        """
                    }
                }
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                sleep 30
                curl -f http://${EC2_HOST}:8180
                '''
            }
        }
    }

    post {

        success {
            echo 'Application deployed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
