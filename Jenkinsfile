pipeline {
    agent any
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '3')) // Keep only last 3 builds to save space
        disableConcurrentBuilds()
        timeout(time: 30, unit: 'MINUTES')
        skipStagesAfterUnstable()
    }
    
    environment {
        MAVEN_OPTS = '-Xmx512m' // Reduce memory usage
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "✅ Code checked out from Git"
            }
        }
        
        stage('Build User Service') {
            steps {
                dir('user-service') {
                    bat 'mvn clean package -DskipTests'
                }
            }
            post {
                success {
                    echo "✅ User Service built successfully"
                }
            }
        }
        
        stage('Build Product Service') {
            steps {
                dir('product-service') {
                    bat 'mvn clean package -DskipTests'
                }
            }
            post {
                success {
                    echo "✅ Product Service built successfully"
                }
            }
        }
        
        stage('Archive Artifacts (Optional)') {
            steps {
                script {
                    // Only archive if space permits, with error handling
                    try {
                        archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true, allowEmptyArchive: true
                        echo "✅ Artifacts archived"
                    } catch (Exception e) {
                        echo "⚠️ Warning: Could not archive artifacts - ${e.message}"
                        echo "💡 Build JARs are still available in workspace"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Clean workspace to free disk space (optional - comment out if you need to keep files)
            // cleanWs()
            
            echo "Build completed - Check workspace for JAR files"
        }
        success {
            echo "🎉 SUCCESS: Both services built successfully!"
            echo "User Service: user-service/target/user-service-0.0.1-SNAPSHOT.jar"
            echo "Product Service: product-service/target/product-service-0.0.1-SNAPSHOT.jar"
        }
        failure {
            echo "❌ FAILURE: Build failed - check logs above"
        }
        unstable {
            echo "⚠️ UNSTABLE: Build completed with warnings"
        }
    }
}
