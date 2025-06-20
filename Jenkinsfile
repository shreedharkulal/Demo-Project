pipeline {
    agent any

    environment {
        IMAGE_NAME = "shreedhar/demo-app"
        IMAGE_TAG = "latest"
        DOCKER_CREDENTIALS_ID = "dockerhub"
        SONARQUBE_ENV = "SonarQube"  // This is the SonarQube server name in Jenkins
    }

    tools {
        maven 'Maven 3'   // Configure Maven in Jenkins (Manage Jenkins > Global Tool Config)
        jdk 'JDK 11'      // Configure JDK 11 in Jenkins
    }

    stages {

        stage('Clone Code') {
            steps {
                git 'https://github.com/your-username/your-repo.git'
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=demo-app -Dsonar.host.url=http://your-sonarqube-server:9000'
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
                script {
                    docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                }
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

