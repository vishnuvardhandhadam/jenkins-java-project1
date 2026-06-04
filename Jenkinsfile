pipeline {
agent any

tools {
    maven 'maven'
}

environment {
    IMAGE_NAME = "vishnuvardhan8328/java-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Checkout') {
        steps {
            git branch: 'main',
                url: 'https://github.com/vishnuvardhandhadam/jenkins-java-project1.git'
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
            docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
            docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
            """
        }
    }

    stage('Docker Hub Login') {
        steps {
            withCredentials([usernamePassword(
                credentialsId: 'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',
                passwordVariable: 'DOCKER_PASS'
            )]) {
                sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                '''
            }
        }
    }

    stage('Push Docker Image') {
        steps {
            sh """
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
            docker push ${IMAGE_NAME}:latest
            """
        }
    }

    stage('Deploy Container') {
        steps {
            sh """
            docker stop java-app || true
            docker rm java-app || true

            docker run -d \
            --name java-app \
            -p 8080:8080 \
            ${IMAGE_NAME}:latest
            """
        }
    }
}

post {
    success {
        echo 'Build Successful'
        echo 'Docker Image Pushed Successfully'
        echo 'Application Deployed Successfully'
    }

    failure {
        echo 'Pipeline Failed'
    }

    always {
        sh 'docker logout || true'
    }
}

}
