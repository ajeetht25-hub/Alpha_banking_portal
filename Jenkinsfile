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
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Check Environment') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Java Version"
                    echo "======================================"
                    java -version

                    echo "======================================"
                    echo "Maven Version"
                    echo "======================================"
                    mvn -version

                    echo "======================================"
                    echo "Node Version"
                    echo "======================================"
                    node --version

                    echo "======================================"
                    echo "NPM Version"
                    echo "======================================"
                    npm --version
                '''
            }
        }

        stage('Backend - Clean') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Cleaning Spring Boot backend...'

                    sh '''
                        mvn clean
                    '''
                }
            }
        }

        stage('Backend - Compile') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Compiling Spring Boot backend...'

                    sh '''
                        mvn compile
                    '''
                }
            }
        }

        stage('Backend - Test') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Running backend tests...'

                    sh '''
                        mvn test
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
                    echo 'Packaging Spring Boot application...'

                    sh '''
                        mvn package -DskipTests
                    '''
                }
            }
        }

        stage('Frontend - Install') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Installing React dependencies...'

                    sh '''
                        npm ci
                    '''
                }
            }
        }

        stage('Frontend - Build') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Building React/Vite application...'

                    sh '''
                        npm run build
                    '''
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'

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
            echo '======================================'
            echo '       BUILD SUCCESSFUL'
            echo '======================================'
        }

        failure {
            echo '======================================'
            echo '          BUILD FAILED'
            echo '======================================'
        }

        always {
            cleanWs()
        }
    }
}
