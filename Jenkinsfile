pipeline {
    agent any

    tools {
        jdk 'JDK-21'
    }

    environment {
        BACKEND_DIR  = 'banking-app'
        FRONTEND_DIR = 'banking-ui'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    java -version
                    ./mvnw -version
                    node --version
                    npm --version
                '''
            }
        }

        stage('Backend - Clean') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh '''
                        chmod +x mvnw
                        ./mvnw clean
                    '''
                }
            }
        }

        stage('Backend - Compile') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh './mvnw compile'
                }
            }
        }

        stage('Backend - Test') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh './mvnw test'
                }
            }

            post {
                always {
                    junit(
                        testResults: 'target/surefire-reports/*.xml',
                        allowEmptyResults: true
                    )
                }
            }
        }

        stage('Backend - Package') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh './mvnw package -DskipTests'
                }
            }
        }

        stage('Frontend - Install') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm ci'
                }
            }
        }

        stage('Frontend - Build') {
            steps {
                dir("${FRONTEND_DIR}") {
                    sh 'npm run build'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                archiveArtifacts(
                    artifacts: '''
                        banking-app/target/*.jar,
                        banking-ui/dist/**
                    ''',
                    fingerprint: true
                )
            }
        }
    }

    post {
        success {
            echo 'BUILD SUCCESSFUL'
        }

        failure {
            echo 'BUILD FAILED'
        }

        always {
            cleanWs()
        }
    }
}
