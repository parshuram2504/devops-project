pipeline {
    agent any
    options {
        skipDefaultCheckout()
    }
environment {
        IMAGE_NAME = "parshuram2504/flask-devops"
    }
    stages {
        stage('Pull Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/parshuram2504/devops-project.git'
                script {
                    def changedFiles = sh(script: 'git diff --name-only HEAD~1 HEAD', returnStdout: true).trim()
                    if (changedFiles == 'deployment.yaml') {
                        currentBuild.result = 'NOT_BUILT'
                        error('Skipping CI - only deployment.yaml was changed')
                    }
                }
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
        stage('Update deployment.yaml') {
            steps {
                script {
                    sh "sed -i 's|image: $IMAGE_NAME:.*|image: $IMAGE_NAME:$BUILD_NUMBER|' deployment.yaml"
                }
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh '''
                        git config user.email "jenkins@ci.com"
                        git config user.name "Jenkins"
                        git add deployment.yaml
                        git commit -m "Update image tag to $BUILD_NUMBER [skip ci]"
                        git push origin main
                    '''
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}

