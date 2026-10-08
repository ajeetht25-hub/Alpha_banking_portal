pipeline {
    agent any

    tools {
        jdk    'JDK-21'
        maven  'Maven-3.9'
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

        stage('Check Environment') {
            steps {
                sh '''
                    echo "===== Java ====="
                    java -version

                    echo "===== Maven ====="
                    mvn -version

                    echo "===== Node ====="
                    node --version

                    echo "===== NPM ====="
                    npm --version
                '''
            }
        }

        stage('Backend - Clean') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn clean'
                }
            }
        }

        stage('Backend - Compile') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn compile'
                }
            }
        }

        stage('Backend - Test') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn test'
                }
            }
            post {
                always {
                    junit(
                        testResults: "${BACKEND_DIR}/target/surefire-reports/*.xml",
                        allowEmptyResults: true
                    )
                }
            }
        }

        stage('Backend - Package') {
            steps {
                dir("${BACKEND_DIR}") {
                    sh 'mvn package -DskipTests'
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
