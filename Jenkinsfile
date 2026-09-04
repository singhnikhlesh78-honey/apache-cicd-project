pipeline {

    agent any

    environment {
        IMAGE_NAME = "nikhleshtest/apache-cicd-newproject"
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    stages {

        stage('Validate') {
            steps {
                sh '''
                    echo "Checking project files..."

                    test -f Dockerfile
                    test -f deployment.yaml
                    test -f service.yaml
                    test -f html/index.html

                    echo "All required files are present."
                '''
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build \
                        -t ${IMAGE_NAME}:${IMAGE_TAG} \
                        -t ${IMAGE_NAME}:latest \
                        .
                '''
            }
        }

        stage('Docker Test') {
            steps {
                sh '''
                    docker rm -f apache-cicd-project-test 2>/dev/null || true

                    docker run -d \
                        --name apache-cicd-project-test \
                        -p 8081:80 \
                        ${IMAGE_NAME}:${IMAGE_TAG}

                    sleep 5

                    curl --fail http://127.0.0.1:8081

                    docker rm -f apache-cicd-project-test
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login \
                            -u "$DOCKER_USER" \
                            --password-stdin

                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker push ${IMAGE_NAME}:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying Apache application to Kubernetes..."

                    kubectl apply -f deployment.yaml
                    kubectl apply -f service.yaml

                    kubectl set image deployment/apache-cicd-project \
                        apache=${IMAGE_NAME}:${IMAGE_TAG}

                    kubectl rollout status deployment/apache-cicd-project --timeout=120s

                    echo "Kubernetes deployment completed."
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Checking Kubernetes deployment..."

                    kubectl get deployment apache-cicd-project

                    kubectl get pods \
                        -l app=apache-cicd-project \
                        -o wide

                    kubectl get service apache-cicd-project

                    kubectl rollout status deployment/apache-cicd-project --timeout=120s

                    echo "Deployment verification completed successfully."
                '''
            }
        }
    }

    post {

        always {
            sh '''
                docker rm -f apache-cicd-project-test 2>/dev/null || true
            '''
        }

        success {
            echo 'Apache Docker + Kubernetes CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Apache Docker + Kubernetes CI/CD pipeline failed!'
        }
    }
}
