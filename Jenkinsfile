pipeline {

    agent any

    environment {
        DOCKERHUB_USER = 'theofuchs'
        IMAGE_TAG = "${env.GIT_COMMIT}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Cast Image') {
            steps {
                sh """
                    docker build \
                      -t ${DOCKERHUB_USER}/cast-service:${IMAGE_TAG} \
                      ./cast-service
                """
            }
        }

        stage('Build Movie Image') {
            steps {
                sh """
                    docker build \
                      -t ${DOCKERHUB_USER}/movie-service:${IMAGE_TAG} \
                      ./movie-service
                """
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh """
                        echo "${DOCKER_PASSWORD}" | docker login \
                          -u "${DOCKER_USER}" \
                          --password-stdin

                        docker push ${DOCKERHUB_USER}/cast-service:${IMAGE_TAG}
                        docker push ${DOCKERHUB_USER}/movie-service:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    helm upgrade --install movie-app ./helm/movie-app \
                      --namespace dev \
                      --create-namespace \
                      --set cast.image.tag=${IMAGE_TAG} \
                      --set movie.image.tag=${IMAGE_TAG}
                """
            }
        }
    }
}
