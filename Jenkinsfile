pipeline {
    agent any

    parameters {
        choice(name: 'ENVIRONMENT', choices: ['Dev', 'Production'], description: 'Deployment Target')
        string(name: 'IMAGE_TAG', defaultValue: 'v1.0', description: 'Docker Image Tag')
    }

    environment {
        DOCKER_HUB_CRED = 'dockerhub-credentials'
        DOCKER_IMAGE    = 'muki1376/disaster-portal'
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
                        echo "Building Docker image ${DOCKER_IMAGE}:${params.IMAGE_TAG}..."
                        bat "docker build -t ${DOCKER_IMAGE}:${params.IMAGE_TAG} ."
                    }
                }
                stage('Verify Artifacts') {
                    steps {
                        echo "Verifying local project files..."
                        bat "if not exist index.html exit /b 1"
                        bat "if not exist deployment.yaml exit /b 1"
                    }
                }
            }
        }

       stage('Publish Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CRED}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    bat 'docker login -u %DOCKER_USER% -p %DOCKER_PASS%'
                    bat "docker push ${DOCKER_IMAGE}:${params.IMAGE_TAG}"
                }
            }
        }

     stage('Deploy to Kubernetes') {
            when {
                expression { return params.ENVIRONMENT == 'Production' }
            }
            steps {
                echo "Deploying application to Kubernetes..."
                bat 'kubectl apply -f deployment.yaml --kubeconfig "C:\\Users\\batch1.VITUNIVERSITY\\.kube\\config"'
            }
        }
    }
}