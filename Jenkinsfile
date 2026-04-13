pipeline {
    agent any

    tools {
        maven 'Maven'
    }

    stages {

        stage('Build User Service') {
            steps {
                echo '========== Building User Service =========='
                dir('user-service') {
                    bat 'mvn clean package -DskipTests'
                }
                echo '========== User Service Build SUCCESS =========='
            }
        }

        stage('Build Product Service') {
            steps {
                echo '========== Building Product Service =========='
                dir('product-service') {
                    bat 'mvn clean package -DskipTests'
                }
                echo '========== Product Service Build SUCCESS =========='
            }
        }

        stage('Run Tests') {
            steps {
                echo '========== Running Tests =========='
                dir('user-service') {
                    bat 'mvn test'
                }
                dir('product-service') {
                    bat 'mvn test'
                }
                echo '========== All Tests PASSED =========='
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '========== Building Docker Images =========='
                dir('user-service') {
                    bat 'docker build -t user-service .'
                }
                dir('product-service') {
                    bat 'docker build -t product-service .'
                }
                echo '========== Docker Images Built Successfully =========='
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '========== Deploying to Kubernetes =========='
                dir('kubernetes') {
                    bat 'kubectl apply -f user-deployment.yaml'
                    bat 'kubectl apply -f product-deployment.yaml'
                }
                echo '========== Deployment Complete =========='
            }
        }

        stage('Verify Deployments') {
            steps {
                echo '========== Verifying Pods =========='
                bat 'kubectl get pods'
                bat 'kubectl get services'
            }
        }

        stage('Show Users and Products') {
            steps {
                echo '========== Fetching Users from User Service =========='
                bat 'curl -s http://localhost:30007/users'

                echo '========== Fetching Products from Product Service =========='
                bat 'curl -s http://localhost:30008/products'

                echo '========== Data Fetch Complete =========='
            }
        }

    }

    post {
        success {
            echo '=========================================='
            echo '   BUILD SUCCESS!'
            echo '   User Service    -> http://localhost:30007/users'
            echo '   Product Service -> http://localhost:30008/products'
            echo '   Both services running in Kubernetes!'
            echo '=========================================='
        }
        failure {
            echo '=========================================='
            echo '   BUILD FAILED - Check logs above'
            echo '=========================================='
        }
    }
}
