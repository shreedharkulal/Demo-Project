pipeline {
    agent {
        docker {
            image 'maven:3.8.8-openjdk-11'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        IMAGE_NAME = "shreedhar/demo-app"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS_ID = "docker-cred"
        SONARQUBE_ENV = "Sonarqube"
        SONAR_HOST_URL = "http://3.108.63.180:9000"
        SONAR_PROJECT_KEY = "demo-app"
    }

    stages {
        stage('Clone Code') {
            steps {
                git 'https://github.com/shreedharkulal/Demo-Project.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh """
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                    -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
            }
        }

        stage('Trivy Scan') {
            steps {
                sh """
                trivy image --exit-code 1 --severity HIGH,CRITICAL ${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${IMAGE_NAME}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Clean Up') {
            steps {
                sh 'docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true'
            }
        }
    }

    post {
        failure {
            echo 'Pipeline failed!'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
    }
}
