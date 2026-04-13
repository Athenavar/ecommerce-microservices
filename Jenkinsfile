pipeline {
    agent any
    
    tools {
        maven 'Maven-3.9'
        jdk 'JDK-21'
    }
    
    environment {
        DOCKER_REGISTRY = 'localhost:5000'
        USER_SERVICE_IMAGE = 'user-service'
        PRODUCT_SERVICE_IMAGE = 'product-service'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo '✅ Source code checked out from GitHub'
            }
        }
        
        stage('Build Services') {
            parallel {
                stage('Build User Service') {
                    steps {
                        dir('user-service') {
                            bat 'mvn clean compile'
                        }
                    }
                }
                stage('Build Product Service') {
                    steps {
                        dir('product-service') {
                            bat 'mvn clean compile'
                        }
                    }
                }
            }
        }
        
        stage('Run Tests') {
            parallel {
                stage('Test User Service') {
                    steps {
                        dir('user-service') {
                            bat 'mvn test'
                        }
                    }
                    post {
                        always {
                            junit 'user-service/target/surefire-reports/*.xml'
                        }
                    }
                }
                stage('Test Product Service') {
                    steps {
                        dir('product-service') {
                            bat 'mvn test'
                        }
                    }
                    post {
                        always {
                            junit 'product-service/target/surefire-reports/*.xml'
                        }
                    }
                }
            }
        }
        
        stage('Package') {
            parallel {
                stage('Package User Service') {
                    steps {
                        dir('user-service') {
                            bat 'mvn package -DskipTests'
                            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                        }
                    }
                }
                stage('Package Product Service') {
                    steps {
                        dir('product-service') {
                            bat 'mvn package -DskipTests'
                            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
                        }
                    }
                }
            }
        }
        
        stage('Build Docker Images') {
            steps {
                script {
                    // Build images
                    bat "docker build -t ${USER_SERVICE_IMAGE}:latest ./user-service"
                    bat "docker build -t ${PRODUCT_SERVICE_IMAGE}:latest ./product-service"
                    
                    // Tag with build number
                    bat "docker tag ${USER_SERVICE_IMAGE}:latest ${USER_SERVICE_IMAGE}:${BUILD_NUMBER}"
                    bat "docker tag ${PRODUCT_SERVICE_IMAGE}:latest ${PRODUCT_SERVICE_IMAGE}:${BUILD_NUMBER}"
                }
                echo '✅ Docker images built'
            }
        }
        
        stage('Push to Registry (Optional)') {
            when {
                branch 'main'
            }
            steps {
                script {
                    // For local registry
                    bat "docker push ${USER_SERVICE_IMAGE}:${BUILD_NUMBER}"
                    bat "docker push ${PRODUCT_SERVICE_IMAGE}:${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                dir('kubernetes') {
                    bat 'kubectl apply -f user-service-deployment.yaml'
                    bat 'kubectl apply -f product-service-deployment.yaml'
                    
                    // Wait for rollout
                    bat 'kubectl rollout status deployment/user-service --timeout=120s'
                    bat 'kubectl rollout status deployment/product-service --timeout=120s'
                }
                echo '✅ Deployed to Kubernetes'
            }
        }
        
        stage('Smoke Tests') {
            steps {
                script {
                    // Wait for services
                    bat 'timeout /t 10 /nobreak'
                    
                    // Test endpoints
                    def userResponse = bat(script: 'curl -s -o nul -w "%{http_code}" http://localhost:30007/actuator/health', returnStdout: true).trim()
                    def productResponse = bat(script: 'curl -s -o nul -w "%{http_code}" http://localhost:30008/actuator/health', returnStdout: true).trim()
                    
                    if (userResponse != '200' || productResponse != '200') {
                        error 'Smoke tests failed!'
                    }
                }
                echo '✅ All services healthy'
            }
        }
        
        stage('Update Terraform State') {
            steps {
                dir('terraform') {
                    bat 'terraform init'
                    bat 'terraform apply -auto-approve'
                }
                echo '✅ Infrastructure state updated'
            }
        }
    }
    
    post {
        always {
            // Clean workspace
            cleanWs()
            
            // Notify
            emailext (
                subject: "Build ${currentBuild.result}: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    Build ${currentBuild.result}
                    Job: ${env.JOB_NAME}
                    Build: ${env.BUILD_NUMBER}
                    Duration: ${currentBuild.durationString}
                    URL: ${env.BUILD_URL}
                """,
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
        
        success {
            echo '🎉 CI/CD Pipeline completed successfully!'
        }
        
        failure {
            echo '❌ Pipeline failed. Check console output.'
        }
    }
}
