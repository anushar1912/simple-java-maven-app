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
        jdk 'java-21'
    }
    
    stages {

        stage('Code Checkout') {
            steps {
                echo '========== Stage 1: Checking out code from Git =========='
                git branch: 'master',
                    credentialsId: 'git-credentials',
                    url: 'https://github.com/THOTASRIHARI506/simple-java-maven-app'
            }
        }

        stage('Build') {
            steps {
                echo '========== Stage 2: Building =========='
                sh 'mvn clean compile'
            }
        }

        stage('SAST - SonarQube') {
            steps {
                echo '========== Running SonarQube =========='
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
                echo '========== Checking Quality Gate =========='
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package Artifact') {
            steps {
                echo '========== Packaging =========='
                sh 'mvn package -DskipTests'
                archiveArtifacts artifacts: "target/${ARTIFACT_NAME}", fingerprint: true
            }
        }

        stage('Deploy to DEV') {
            steps {
                echo 'Deploying to DEV'
                sh """
                    mkdir -p /tmp/deployments/dev
                    cp target/${ARTIFACT_NAME} /tmp/deployments/dev/
                """
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo 'Deploying to UAT'
                sh """
                    mkdir -p /tmp/deployments/uat
                    cp target/${ARTIFACT_NAME} /tmp/deployments/uat/
                """
            }
        }

        stage('Approval for PROD') {
            steps {
                input message: "Approve PROD deployment?", ok: "Deploy"
            }
        }

        stage('Deploy to PROD') {
            steps {
                echo 'Deploying to PROD'
                sh """
                    mkdir -p /tmp/deployments/prod
                    cp target/${ARTIFACT_NAME} /tmp/deployments/prod/
                """
            }
        }

        stage('Health Check Simulation') {
            steps {
                echo 'Running health checks'
                sh """
                    echo 'Health Check: OK'
                    echo 'Deployment successful'
                """
            }
        }
    }

    post {
        success {
            echo '========== Pipeline Completed Successfully =========='
            emailext(
                subject: "SUCCESS: Jenkins Pipeline - ${APP_NAME} #${BUILD_NUMBER}",
                body: """
                    Pipeline executed successfully!
                    
                    Job: ${JOB_NAME}
                    Build: ${BUILD_NUMBER}
                    Artifact: ${ARTIFACT_NAME}
                    
                    View details: ${BUILD_URL}
                """,
                to: 'test@example.com'
            )
        }
        
        failure {
            echo '========== Pipeline Failed =========='
            emailext(
                subject: "FAILURE: Jenkins Pipeline - ${APP_NAME} #${BUILD_NUMBER}",
                body: """
                    Pipeline execution failed!
                    
                    Job: ${JOB_NAME}
                    Build: ${BUILD_NUMBER}
                    
                    View logs: ${BUILD_URL}console
                """,
                to: 'test@example.com'
            )
        }
        
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}




