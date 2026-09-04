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
                        credentialsId: 'dockerhub',
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
    }

    post {

        always {
            sh '''
                docker rm -f apache-cicd-project-test 2>/dev/null || true
            '''
        }

        success {
            echo 'Apache Docker CI pipeline completed successfully!'
        }

        failure {
            echo 'Apache Docker CI pipeline failed!'
        }
    }
}
