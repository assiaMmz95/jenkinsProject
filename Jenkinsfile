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
       /* stage('build'){
            steps {
                bat './mvnw package'
                archiveArtifacts 'target/*.jar'
            }*/
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
       // }
        stage('deploy'){
              when{
                branch 'main'
              }
              steps {
                  bat 'docker-compose up --build -d'
              }
        }
        stage('Health Check') {
            steps {
                echo "Checking Health..."
                sleep time: 10, unit: 'SECONDS'


                script {

                    def result = bat(
                        '''
                            @echo off
                            curl -s --connect-timeout 5 --max-time 10 ^
                                -o response.json ^
                                -w "%%{http_code}" ^
                                http://localhost:8082/actuator/health
                            if errorlevel 1 echo 000
                        ''',
                        returnStdout: true
                    ).trim()


                    def httpCode = result


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
                }
            }
        }

    }
}