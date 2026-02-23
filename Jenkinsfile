pipeline{
    agent any
    stages{
         stage('init'){
            steps {
                bat './mvnw clean'
            }
        }
         /* stage('test'){
            steps {
                bat './mvnw test'
                junit 'target/surefire-reports *//*.xml'
            }
        } */
        /* stage('documentation'){
            steps {
                bat './mvnw javadoc:javadoc'
                publishHTML ([
                 allowMissing: false,
                 alwaysLinkToLastBuild: true,
                 keepAll: true,
                 reportDir: 'target/site/apidocs',
                 reportFiles: 'index.html',
                 reportName: 'Documentation'
                 ])
            }
        } */
        stage('build'){
            steps {
                bat './mvnw package'
                archiveArtifacts 'target/*.jar'
            }
            /* post{
                 *//* always{
                    emailext(subject: "Build réussi:",
                            body:"Le build a réussi."
                            to: "rina.ra.1804@gmail.com"
                            )
                    } *//*
                failure{
                        mail(subject: "Build echec:",
                                body:"Le build a réussi.",
                                to: "rina.ra.1804@gmail.com"
                                )
                }
                success{
                        mail(subject: "Build réussi:",
                                    body:"Le build a réussi.",
                                    to: "rina.ra.1804@gmail.com"
                                    )
                }
            } */


        }
        stage('deploy'){
              when{
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
                sleep time: 15, unit: 'SECONDS'

              script {
                    def result = bat(script: '''
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

                       if (httpCode == "200") {


                           def body = readFile('response.json')
                           echo "Body: ${body}"


                           if (body.contains('"status":"UP"')) {
                               echo "Application is healthy ✅"
                           } else {
                                currentBuild.result = 'FAILURE'
                           }


                       } else {
                           echo "Application not reachable"
                            currentBuild.result = 'FAILURE'

                       }
                         echo "Result: ${currentBuild.result}"
                   }
               }
            }

        stage('Rollback') {
            when {
                expression { currentBuild.result == "FAILURE" }
            }
            steps {
                /*  def stableTag = sh(
                         script: "git tag --sort=-creatordate | head -n 1",
                         returnStdout: true
                     ).trim()
                 echo "pro stable ${stableTag}" */
                echo "Starting rollback to tag: ${ROLLBACK_TAG}"
                script {
                   bat """
                        git fetch origin --tags --force
                        git checkout tags/${ROLLBACK_TAG} -b ${ROLLBACK_BRANCH}
                    """

                    echo "Rolled back to tag ${ROLLBACK_TAG} on new branch ${ROLLBACK_BRANCH}"


                    //sh './deploy.sh'
                    bat './mvnw clean'
                    bat './mvnw install'

                    // Stop and remove containers safely
                    bat 'docker-compose down --remove-orphans'
                    bat 'docker rm -f spring-boot-app || exit 0'
                    bat 'docker rm -f mysql-db || exit 0'

                    // Rebuild and start
                    bat 'docker-compose up --build -d'

                    echo "Rollback deployment complete"




                }
            }
        }

     /*   stage('Rollback') {
           when {
               expression { currentBuild.result == 'FAILURE' }
           }
           steps {
              
               echo "Starting rollback to tag: ${ROLLBACK_TAG}"
               script {
                   sh """
                        git fetch origin --tags --force                                        git checkout tags/${ROLLBACK_TAG} -b ${ROLLBACK_BRANCH}
                   """
                   echo "Rolled back to tag ${ROLLBACK_TAG} on new branch ${ROLLBACK_BRANCH}"


                   bat 'clean package'
                   bat 'docker-compose up --build -d'
                   echo "Rollback deployment complete"




               }
           }
       }

        environment {
            ROLLBACK_TAG = "v1.0.0"   // Set your stable rollback tag here
            ROLLBACK_BRANCH = "rollback/hotfix-1.0.0"
        }*/


    }
}