pipeline {
    agent any
    
    environment {
        APP_NAME = 'my-application'
        VERSION = "${BUILD_NUMBER}"
        ARTIFACT_NAME = "${APP_NAME}-${VERSION}.jar"
        SONAR_PROJECT_KEY = 'my-app-key'
    }
    
    tools {
        maven 'Maven'
        jdk 'JDK-21'
    }
    
    stages {
        stage('Code Checkout') {
            steps {
                echo '========== Stage 1: Checking out code from Git =========='
                git branch: 'main',
                    credentialsId: 'git-credentials',
                    url: 'https://github.com/anushar1912/jenkines-test'
            }
        }
        
        stage('Build') {
            steps {
                echo '========== Stage 2: Building the application =========='
                sh 'mvn clean compile'
            }
        }

        /*
        ===============================
        SONARQUBE STAGES (DISABLED)
        ===============================

        stage('SAST - SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis'
                withSonarQubeEnv('SonarQube') {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.projectName=${APP_NAME} \
                          -Dsonar.projectVersion=${VERSION}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        */

        stage('Artifact Packaging') {
            steps {
                echo 'Packaging artifact'
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }
        
        stage('Deployment Simulation') {
            steps {
                echo 'Simulated deployment'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
