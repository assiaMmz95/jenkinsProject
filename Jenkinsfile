pipeline{
agent any
stages{
stage('test'){
steps {
junit 'target/surefire-reports/*.xml'
}
}
stage('build'){
steps {
bat './mvnw clean package'
archiveArtifacts 'target/*.jar'
}
}
}
}