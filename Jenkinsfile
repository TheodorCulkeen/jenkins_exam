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

        stage('Deploy-dev') {
            steps {
                sh """
                    helm upgrade --install movie-app ./k3s/movie-app \
                      --namespace dev \
                      --create-namespace \
                      --set cast.image.tag=${IMAGE_TAG} \
                      --set movie.image.tag=${IMAGE_TAG} \
                      --set nginx.service.nodePort=30087
                """
            }
        }
        
        stage('Deploy-qa') {
            steps {
                sh """
                    helm upgrade --install movie-app ./k3s/movie-app \
                      --namespace qa \
                      --create-namespace \
                      --set cast.image.tag=${IMAGE_TAG} \
                      --set movie.image.tag=${IMAGE_TAG} \
                      --set nginx.service.nodePort=30086
                """
            }
        }

        
        stage('Deploy-staging') {
            steps {
                sh """
                    helm upgrade --install movie-app ./k3s/movie-app \
                      --namespace staging \
                      --create-namespace \
                      --set cast.image.tag=${IMAGE_TAG} \
                      --set movie.image.tag=${IMAGE_TAG} \
                      --set nginx.service.nodePort=30090
                """
            }
        }
            
        stage('Deploy Production') {
          when {
        	expression {
            	env.GIT_BRANCH == 'origin/main'
        	}
    	}
        
            steps {                    
            	input message: 'Deploy this build to production?',
            		ok: 'Deploy'        		
                        
                sh """
                    helm upgrade --install movie-app-prod ./k3s/movie-app \
                      --namespace prod \
                      --create-namespace \
                      -f ./helm/values-prod.yaml \
                      --set cast.image.tag=${IMAGE_TAG} \
                      --set movie.image.tag=${IMAGE_TAG} \
                      --set nginx.service.nodePort=30094

                """
            }
        }                                    
    }
}
