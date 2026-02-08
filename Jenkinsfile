pipeline{
agent any
stages{
stage('build'){
steps {
bat './mvnw clean package'
archiveArtifacts 'target/*.jar'
junit 'target/surefire-reports/*.xml'
}
}
}
}