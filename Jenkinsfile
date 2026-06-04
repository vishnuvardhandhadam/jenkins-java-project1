pipeline {
agent any


tools {
    maven 'Maven'
}

environment {
    IMAGE_NAME = "vishnuvardhan8328/java-app"
    IMAGE_TAG = "${BUILD_NUMBER}"
}

stages {

    stage('Checkout') {
        steps {
            git branch: 'main', url: 'https://github.com/vishnuvardhandhadam/jenkins-java-project1.git'
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
}


}
