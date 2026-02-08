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
stage('build'){
steps {
bat './mvnw package'
archiveArtifacts 'target/*.jar'
}
}
stage('documentation'){
steps {
bat './mvnw javadoc:javadoc'
bat '''
mkdir -p documentationcp -r
target/site/*
zip -r doc.zip doc
'''
archiveArtifacts artifacts : 'doc.zip'
}
}
}
}