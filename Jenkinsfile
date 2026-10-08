pipeline {
    agent any

    tools {
        jdk 'JDK-17'
        nodejs 'NodeJS-20'
    }

    environment {
        BACKEND_DIR  = 'banking-app'
        FRONTEND_DIR = 'banking-ui'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Backend - Clean') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Cleaning backend project...'

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
                    echo 'Compiling Spring Boot backend...'

                    sh '''
                        ./mvnw compile
                    '''
                }
            }
        }

        stage('Backend - Test') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Running backend tests...'

                    sh '''
                        ./mvnw test
                    '''
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
                    echo 'Creating backend JAR...'

                    sh '''
                        ./mvnw package -DskipTests
                    '''
                }
            }
        }

        stage('Frontend - Install') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Installing frontend dependencies...'

                    sh '''
                        npm ci
                    '''
                }
            }
        }

        stage('Frontend - Test') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Running frontend tests...'

                    sh '''
                        npm test -- --run
                    '''
                }
            }
        }

        stage('Frontend - Build') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Building React/Vite frontend...'

                    sh '''
                        npm run build
                    '''
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving application artifacts...'

                archiveArtifacts(
                    artifacts: '''
                        banking-app/target/*.jar,
                        banking-ui/dist/**
                    ''',
                    fingerprint: true,
                    allowEmptyArchive: false
                )
            }
        }
    }

    post {

        success {
            echo '=========================================='
            echo '       BUILD SUCCESSFUL'
            echo '=========================================='
        }

        failure {
            echo '=========================================='
            echo '          BUILD FAILED'
            echo '=========================================='
        }

        always {
            echo 'Cleaning Jenkins workspace...'
            cleanWs()
        }
    }
}
