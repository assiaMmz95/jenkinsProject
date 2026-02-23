pipeline {
    agent any
    environment {
        ROLLBACK_TAG = "v1.0.0"   // Set your stable rollback tag here
        ROLLBACK_BRANCH = "rollback/hotfix-1.0.0"
    }
    stages {
        stage('init') {
            steps {
                bat './mvnw clean'
            }
        }
        
        stage('build') {
            steps {
                bat './mvnw package'
                archiveArtifacts 'target/*.jar'
            }
        }
        
        stage('deploy') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying application...'

                // Stop and remove containers safely
                bat 'docker-compose down --remove-orphans'
                bat 'docker rm -f spring-boot-app || exit 0'
                bat 'docker rm -f mysql-db || exit 0'

                // Rebuild and start
                bat 'docker-compose up --build -d'
            }
        }

        stage('Health Check') {
            steps {
                echo "Checking Health..."
                sleep time: 30, unit: 'SECONDS'  // Increased from 15 to 30 seconds

                script {
                    def maxRetries = 3
                    def success = false
                    
                    for (int i = 1; i <= maxRetries; i++) {
                        echo "Health check attempt ${i}/${maxRetries}"
                        
                        try {
                            // Clean up previous files
                            bat 'if exist response.json del response.json'
                            bat 'if exist status.txt del status.txt'
                            
                            def result = bat(
                                script: '''
                                    @echo off
                                    setlocal
                                
                                    curl -s -o response.json -w "%%{http_code}" http://localhost:8082/actuator/health > status.txt 2>nul
                                
                                    if errorlevel 1 (
                                        echo 000 > status.txt
                                    )
                                
                                    set /p code=<status.txt
                                    echo %code%
                                
                                    exit /b 0
                                ''',
                                returnStdout: true
                            ).trim()

                            def httpCode = result[-3..-1]
                            echo "HTTP Code: ${httpCode}"

                            if (httpCode == "200" && fileExists('response.json')) {
                                def body = readFile('response.json')
                                echo "Body: ${body}"

                                if (body.contains('"status":"UP"')) {
                                    echo "Application is healthy ✅"
                                    success = true
                                    break
                                } else {
                                    echo "Application returned non-UP status"
                                }
                            } else {
                                echo "Application not reachable (HTTP: ${httpCode})"
                            }
                        } catch (Exception e) {
                            echo "Error during health check: ${e.getMessage()}"
                        }
                        
                        if (i < maxRetries) {
                            echo "Waiting 10 seconds before next attempt..."
                            sleep time: 10, unit: 'SECONDS'
                        }
                    }
                    
                    if (!success) {
                        currentBuild.result = 'FAILURE'
                        error "Health check failed after ${maxRetries} attempts"
                    }
                    
                    echo "Final Result: ${currentBuild.result}"
                }
            }
        }

        stage('Rollback') {
            when {
                expression { currentBuild.result == "FAILURE" }
            }
            steps {
                script {
                    echo "Starting rollback process..."
                    
                    // Use the environment variable or fallback to default
                    def rollbackTag = env.ROLLBACK_TAG ?: "v1.0.0"
                    def rollbackBranch = "rollback-${rollbackTag}-${BUILD_NUMBER}"
                    
                    echo "Starting rollback to tag: ${rollbackTag}"
                    
                    // Fetch all tags and checkout
                    bat """
                        git fetch origin --tags --force
                        git checkout tags/${rollbackTag} -b ${rollbackBranch}
                    """

                    echo "Rolled back to tag ${rollbackTag} on new branch ${rollbackBranch}"

                    // Clean and build
                    bat './mvnw clean'
                    bat './mvnw install'

                    // Stop and remove containers safely
                    bat 'docker-compose down --remove-orphans'
                    bat 'docker rm -f spring-boot-app || exit 0'
                    bat 'docker rm -f mysql-db || exit 0'

                    // Rebuild and start
                    bat 'docker-compose up --build -d'

                    echo "Rollback deployment complete"
                    
                    // Verify rollback with health check
                    sleep time: 30, unit: 'SECONDS'
                    
                    def healthCheck = bat(
                        script: 'curl -s -o response.json -w %%{http_code} http://localhost:8082/actuator/health',
                        returnStdout: true,
                        returnStatus: true
                    )
                    
                    if (healthCheck == 0 && fileExists('response.json')) {
                        def httpCode = readFile('response.json') ?: "000"
                        if (httpCode.contains("200")) {
                            echo "Rollback successful - application is healthy"
                        } else {
                            error "Rollback failed - application returned HTTP ${httpCode}"
                        }
                    } else {
                        error "Rollback failed - application health check failed"
                    }
                }
            }
        }
    }
    
    // Post section must be OUTSIDE the stages block
    post {
        success {
            echo 'Pipeline completed successfully ✅'
        }
        failure {
            echo 'Pipeline failed ❌'
        }
        always {
            // Clean up temporary files
            script {
                bat 'if exist response.json del response.json'
                bat 'if exist status.txt del status.txt'
            }
        }
    }
}
