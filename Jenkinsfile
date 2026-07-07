pipeline {
    agent any

    tools {
        jdk 'jdk11'
        maven 'maven3'
    }

    environment {
        IMAGE_NAME = "battyfr/boardgame:latest"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=Boardgame \
                        -Dsonar.projectName=Boardgame
                    '''
                }
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy to Nexus') {
            steps {
                configFileProvider([
                    configFile(
                        fileId: 'maven-settings',
                        variable: 'MAVEN_SETTINGS'
                    )
                ]) {
                    sh 'mvn deploy -s $MAVEN_SETTINGS'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t $IMAGE_NAME .
                '''
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                        docker push $IMAGE_NAME
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    kind load docker-image $IMAGE_NAME --name boardgame
                    kubectl apply -f k8s/
                    kubectl rollout restart deployment/boardgame
                    kubectl rollout status deployment/boardgame
                '''
            }
        }
    }

    post {

        success {
            echo '========================================='
            echo ' CI/CD Pipeline Completed Successfully!'
            echo '========================================='
            echo '✓ Source Code Checked Out'
            echo '✓ Maven Compile Successful'
            echo '✓ Unit Tests Passed'
            echo '✓ SonarQube Analysis Completed'
            echo '✓ JAR Package Created'
            echo '✓ Artifact Uploaded to Nexus'
            echo '✓ Docker Image Built'
            echo '✓ Docker Image Pushed to Docker Hub'
            echo '✓ Kubernetes Deployment Updated'
            echo '========================================='
        }

        failure {
            echo '========================================='
            echo ' Pipeline Failed!'
            echo 'Please check the failed stage above.'
            echo '========================================='
        }

        always {
            cleanWs()
        }
    }
}
