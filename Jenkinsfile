pipeline {
    agent any

    environment {
        DOCKERHUB_USER = '93anil'
        IMAGE_NAME     = 'my-note-app'
        IMAGE_TAG      = 'microdegree'
        FULL_IMAGE     = "${DOCKERHUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
    }

    stages {

        stage('Code Cloning') {
            steps {
                echo 'Cloning the code'
                git branch: 'main',
                    url: 'https://github.com/93Anil/django-notes-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'
                sh '''
                  docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                echo 'Tagging Docker image'
                sh '''
                  docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}
                '''
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                echo 'Pushing image to DockerHub'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerHub',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    sh '''
                      echo "$DOCKERHUB_PASSWORD" | docker login -u "$DOCKERHUB_USERNAME" --password-stdin
                      docker push ${FULL_IMAGE}
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying to Kubernetes'
                withKubeConfig(credentialsId: 'kubeconfig') {
                    sh '''
                      kubectl apply -f notesapp/deployment.yaml
                      kubectl apply -f notesapp/service.yaml
                      kubectl rollout status deployment todo-deployment
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully 🎉'
        }
        failure {
            echo 'Pipeline failed ❌'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
