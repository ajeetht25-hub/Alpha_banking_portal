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
                    echo "===== JAVA ====="
                    java -version

                    echo "===== MAVEN ====="
                    if ! command -v mvn >/dev/null 2>&1; then
                        echo "ERROR: Maven is not installed."
                        exit 1
                    fi
                    mvn -version

                    echo "===== NODE ====="
                    if ! command -v node >/dev/null 2>&1; then
                        echo "ERROR: Node.js is not installed."
                        exit 1
                    fi
                    node --version

                    echo "===== NPM ====="
                    if ! command -v npm >/dev/null 2>&1; then
                        echo "ERROR: npm is not installed."
                        exit 1
                    fi
                    npm --version
                '''
            }
        }

        stage('Backend - Clean') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Cleaning backend...'
                    sh 'mvn clean'
                }
            }
        }

        stage('Backend - Compile') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Compiling backend...'
                    sh 'mvn compile'
                }
            }
        }

        stage('Backend - Test') {
            steps {
                dir("${BACKEND_DIR}") {
                    echo 'Running backend tests...'
                    sh 'mvn test'
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
                    echo 'Packaging backend...'
                    sh 'mvn package -DskipTests'
                }
            }
        }

        stage('Frontend - Install') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Installing frontend dependencies...'
                    sh 'npm ci'
                }
            }
        }

        stage('Frontend - Build') {
            steps {
                dir("${FRONTEND_DIR}") {
                    echo 'Building frontend...'
                    sh 'npm run build'
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving artifacts...'

                archiveArtifacts(
                    artifacts: 'banking-app/target/*.jar, banking-ui/dist/**',
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
