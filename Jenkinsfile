pipeline {
    agent any

    tools {
        maven 'Maven3'
    }

    environment {
        SONAR_HOST_URL = 'http://http://13.201.172.136/:9000'
        NEXUS_URL = 'http://13.200.39.65/:8081'
        NEXUS_CREDENTIALS = 'nexus-credentials'
        DOCKER_IMAGE = 'petclinic'
        DOCKER_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Git Checkout') {
            steps {
                echo '========== Pulling Code from GitHub =========='
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                echo '========== Building with Maven =========='
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Unit Tests') {
            steps {
                echo '========== Running Unit Tests =========='
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo '========== Scanning Code Quality =========='
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo '========== Checking Quality Gate =========='
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to Nexus') {
            steps {
                echo '========== Uploading Artifact to Nexus =========='
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_URL}",
                    groupId: 'org.springframework.samples',
                    version: "${BUILD_NUMBER}",
                    repository: 'maven-snapshots',
                    credentialsId: "${NEXUS_CREDENTIALS}",
                    artifacts: [
                        [
                            artifactId: 'spring-petclinic',
                            classifier: '',
                            file: "target/spring-petclinic-*.jar",
                            type: 'jar'
                        ]
                    ]
                )
            }
        }

        stage('Docker Build') {
            steps {
                echo '========== Building Docker Image =========='
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }

        stage('Docker Run') {
            steps {
                echo '========== Running Docker Container =========='
                sh "docker stop petclinic || true"
                sh "docker rm petclinic || true"
                sh "docker run -d --name petclinic -p 8090:8080 ${DOCKER_IMAGE}:${DOCKER_TAG}"
            }
        }

    }

    post {
        success {
            echo '========== ✅ Pipeline Succeeded! App is Live! =========='
        }
        failure {
            echo '========== ❌ Pipeline Failed! Check logs above =========='
        }
    }
}
