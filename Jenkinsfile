pipeline {
    agent any
    triggers { pollSCM('H/5 * * * *') }
    tools { maven 'Maven3' }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Build') { steps { sh 'mvn clean package -DskipTests' } }
        stage('Unit Tests') { steps { sh 'mvn test' } }
    }
}