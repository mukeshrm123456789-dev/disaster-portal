pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['Dev', 'Production'], description: 'Deployment Target')
        string(name: 'IMAGE_TAG', defaultValue: 'v1.0', description: 'Docker Image Tag')
    }

    environment {
        DOCKER_HUB_CRED = 'dockerhub-credentials'
        DOCKER_IMAGE    = '<YOUR_DOCKERHUB_USERNAME>/disaster-portal'
    }

    stages {
        stage('Checkout & Lint') {
            steps {
                echo 'Checking out source repository...'
                checkout scm
            }
        }

        stage('Parallel Build & Test') {
            parallel {
                stage('Docker Build') {
                    steps {
                        sh "docker build -t ${DOCKER_IMAGE}:${params.IMAGE_TAG} ."
                    }
                }
                stage('Verify Artifacts') {
                    steps {
                        sh 'test -f index.html'
                    }
                }
            }
        }

        stage('Publish Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                    sh "docker push ${DOCKER_IMAGE}:${params.IMAGE_TAG}"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { return params.ENVIRONMENT == 'Production' }
            }
            steps {
                sh """
                  sed -i 's|<muki1376>/disaster-portal:.*|${DOCKER_IMAGE}:${params.IMAGE_TAG}|g' deployment.yaml
                  kubectl apply -f deployment.yaml
                """
            }
        }
    }
}