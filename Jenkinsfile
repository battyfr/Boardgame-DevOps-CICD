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
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
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
                configFileProvider([configFile(fileId: 'maven-settings', variable: 'MAVEN_SETTINGS')]) {
                    sh 'mvn deploy -s $MAVEN_SETTINGS'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
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
    }

    post {

        success {
            echo '========================================='
            echo 'Pipeline Completed Successfully!'
            echo 'GitHub ✓'
            echo 'Maven Build ✓'
            echo 'Tests Passed ✓'
            echo 'SonarQube ✓'
            echo 'Nexus Deployment ✓'
            echo 'Docker Image Built ✓'
            echo 'Docker Hub Push ✓'
            echo '========================================='
        }

        failure {
            echo 'Pipeline Failed!'
        }

        always {
            cleanWs()
        }
    }
}
