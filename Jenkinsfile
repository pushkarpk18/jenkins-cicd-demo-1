pipeline {
    agent any

    tools {
        maven 'Maven3'
        jdk 'JDK17'
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        DOCKER_IMAGE = "pushkarkumbharepk18/demo-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
        APP_SERVER_IP = "107.22.118.78"
    }

    stages {

        stage('Git Clone') {
            steps {
                echo 'Cloning repository from GitHub...'
                git branch: 'main',
                    url: 'https://github.com/pushkarpk18/jenkins-cicd-demo.git'
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Building application with Maven...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Maven Test') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image...'
                sh "docker build -t ${DOCKER_IMAGE}:${IMAGE_TAG} -t ${DOCKER_IMAGE}:latest ."
            }
        }

        stage('Docker Login & Push') {
            steps {
                echo 'Logging into Docker Hub and pushing image...'
                sh '''
                    echo "$DOCKERHUB_CREDENTIALS_PSW" | docker login -u "$DOCKERHUB_CREDENTIALS_USR" --password-stdin
                    docker push ${DOCKER_IMAGE}:${IMAGE_TAG}
                    docker push ${DOCKER_IMAGE}:latest
                '''
            }
        }

        stage('Deploy to App Server') {
            steps {
                echo 'Deploying container to App Server via SSH...'
                sshagent(credentials: ['app-server-ssh-key']) {
                    sh """
                        ssh -o StrictHostKeyChecking=no ubuntu@${APP_SERVER_IP} '
                            docker pull ${DOCKER_IMAGE}:latest &&
                            docker stop demo-app || true &&
                            docker rm demo-app || true &&
                            docker run -d --name demo-app -p 8080:8080 ${DOCKER_IMAGE}:latest
                        '
                    """
                }
            }
        }

        stage('Cleanup Local Images') {
            steps {
                echo 'Cleaning up dangling local images on Jenkins server...'
                sh 'docker image prune -f'
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline completed successfully! App deployed to App Server.'
        }
        failure {
            echo '❌ Pipeline failed. Check the stage logs above.'
        }
        always {
            sh 'docker logout || true'
        }
    }
}
