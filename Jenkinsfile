pipeline {
    agent any
    triggers { pollSCM('H/5 * * * *') }
    tools { maven 'Maven3' }
    stages {
        stage('Checkout') { steps { checkout scm } }
        stage('Build') { steps { sh 'mvn clean package -DskipTests' } }
        stage('Unit Tests') { steps { sh 'mvn test' } }

        // SonarQube static analysis
        stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            // explicitly specify project key so it's correctly using project-specific token
            sh 'mvn sonar:sonar -Dsonar.projectKey=spring-petclinic -Dsonar.projectName=spring-petclinic'
        }
    }
}
stage('Quality Gate') {
    steps {
        timeout(time: 5, unit: 'MINUTES') {
            waitForQualityGate abortPipeline: true
        }
    }
}
    }
}