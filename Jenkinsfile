pipeline {
    agent {
        label 'docker-host'
    }

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-21-openjdk-amd64'
        PATH+JDK21 = '/usr/lib/jvm/java-21-openjdk-amd64/bin'

        REGISTRY = 'docker-host:5000'
        IMAGE_NAME = 'richard-petstore'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/RichardAkbar/devsecops-3-tier-cicd-pipeline.git'
            }
        }

        stage('Verify Environment') {
            steps {
                sh '''
                    echo "===== JAVA ====="
                    java -version
                    javac -version

                    echo "===== MAVEN ====="
                    mvn -version

                    echo "===== DOCKER ====="
                    docker --version

                    echo "===== TRIVY ====="
                    trivy --version
                '''
            }
        }

        stage('Build and Test') {
            steps {
                dir('richard-petstore') {
                    sh 'mvn clean verify'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                dir('richard-petstore') {
                    withSonarQubeEnv('SonarQubeCloud') {
                        sh '''
                            mvn org.sonarsource.scanner.maven:sonar-maven-plugin:5.8.0.7211:sonar \
                              -Dsonar.organization=richardakbar \
                              -Dsonar.projectKey=RichardAkbar_devsecops-3-tier-cicd-pipeline \
                              -Dsonar.host.url=https://sonarcloud.io \
                              -Dsonar.qualitygate.wait=true \
                              -Dsonar.qualitygate.timeout=300
                        '''
                    }
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('richard-petstore') {
                    sh '''
                        docker build \
                          -t ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER} \
                          -t ${REGISTRY}/${IMAGE_NAME}:latest .
                    '''
                }
            }
        }

        stage('Trivy Security Scan') {
            steps {
                sh '''
                    echo "===== TRIVY VULNERABILITY SCAN ====="

                    trivy image \
                      --scanners vuln \
                      ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}

                    echo "===== SECURITY GATE: HIGH AND CRITICAL ====="

                    trivy image \
                      --scanners vuln \
                      --severity HIGH,CRITICAL \
                      --exit-code 1 \
                      ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'docker-registry-credentials',
                        usernameVariable: 'REGISTRY_USER',
                        passwordVariable: 'REGISTRY_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$REGISTRY_PASSWORD" | docker login "$REGISTRY" \
                          -u "$REGISTRY_USER" --password-stdin

                        docker push ${REGISTRY}/${IMAGE_NAME}:${BUILD_NUMBER}
                        docker push ${REGISTRY}/${IMAGE_NAME}:latest

                        docker logout "$REGISTRY"
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Richard PetStore CI/CD pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the console output.'
        }
    }
}
