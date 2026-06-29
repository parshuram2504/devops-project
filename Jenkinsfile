pipeline {
    agent any
environment {
        IMAGE_NAME = "parshuram2504/flask-devops"
    }
    stages {
        stage('Pull Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/parshuram2504/devops-project.git'
            }
        }
        stage('SonarQube Analysis') {
    steps {
        script {
            def scannerHome = tool 'sonar-scanner'

            withSonarQubeEnv('sonarqube') {
                sh """
                ${scannerHome}/bin/sonar-scanner \
                -Dsonar.projectKey=flask-app \
                -Dsonar.sources=. \
                -Dsonar.python.version=3.11 \
                -Dsonar.host.url=http://172.20.243.147:9000
                """
            }
        }
    }
}
        stage('Docker Build') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }
        stage('Trivy Scan') {
            steps {
                sh 'trivy image $IMAGE_NAME:$BUILD_NUMBER'
            }
        }
        stage('Docker Push') {
    steps {
        withDockerRegistry(
            credentialsId: 'dockerhub',
            url: 'https://index.docker.io/v1/'
        ) {
            sh 'docker push $IMAGE_NAME:$BUILD_NUMBER'
        }
    }
}
    }
}

