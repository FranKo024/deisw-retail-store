pipeline {
    agent any
    tools {
        maven 'MAVEN_3_9'
        jdk 'JDK_26'
    }
    environment {
        REGISTRY_USER = "u202311959"
        IMAGE_NAME    = "retail-store-U202311959"   // retail-store-<tu código>
        TAG           = "${env.BUILD_NUMBER}"
    }
    stages {
        stage('1. Compile Project') {
            steps { sh 'mvn -B clean compile' }
        }
        stage('2. Unit Tests') {
            steps { sh 'mvn -B test' }
        }
        stage('3. SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MiSonarServer') {
                    sh "mvn -B clean verify sonar:sonar -Dsonar.projectKey=${IMAGE_NAME}"
                }
            }
        }
        stage('4. Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('5. Construir y Publicar Imagen Docker') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_CREDENTIALS',
                        usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo '${DOCKER_PASS}' | docker login -u '${DOCKER_USER}' --password-stdin"
                    sh "docker buildx build --platform linux/amd64 -t ${REGISTRY_USER}/${IMAGE_NAME}:${TAG} -t ${REGISTRY_USER}/${IMAGE_NAME}:latest --push ."
                }
            }
        }
    }
}
