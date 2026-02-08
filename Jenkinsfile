pipeline{
    agent any
    stages{
        stage('init'){
            steps {
                bat './mvnw clean'
            }
        }
        stage('test'){
            steps {
                bat './mvnw test'
                junit 'target/surefire-reports/*.xml'
            }
        }
        stage('documentation'){
            steps {
                bat './mvnw javadoc:javadoc'
                /* bat '''
                mkdir -p documentationcp -r
                target/site *//*
                zip -r doc.zip doc
                '''
                archiveArtifacts artifacts : 'doc.zip' */
                publishHTML ([
                 allowMissing: false,
                 alwaysLinkToLastBuild: true,
                 keepAll: true,
                 reportDir: 'target/site/apidocs',
                 reportFiles: 'index.html',
                 reportName: 'Documentation'
                 ])
            }
        }
        stage('build'){
            steps {
                bat './mvnw package'
                archiveArtifacts 'target/*.jar'
            }
            //post{
                /* always{
                    emailext(subject: "Build réussi:",
                            body:"Le build a réussi."
                            to: "rina.ra.1804@gmail.com"
                            )
                    } */
                failure{
                    emailext(subject: "Build echec:",
                            body:"Le build a réussi.",
                            to: "rina.ra.1804@gmail.com"
                            )
                }
                success{
                        emailext(subject: "Build réussi:",
                                body:"Le build a réussi.",
                                to: "rina.ra.1804@gmail.com"
                                )
                }
           // }
        }

    }
}