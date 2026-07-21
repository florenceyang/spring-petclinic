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

        // ZAP Security Scan
        // remove any pre-existing, create new zap container that's mounted
        // to /zap/wrk (to be able to generate zap report)
        stage('Start ZAP') {
            steps {
                sh '''
                docker rm -f zap || true
                docker run -d --name zap --network miniproject-network \
                    -v $(pwd):/zap/wrk:rw --user root \
                    zaproxy/zap-stable sleep infinity
                '''
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh '''
                ./mvnw -q spring-boot:build-image -Dspring-boot.build-image.imageName=petclinic:staging
                docker rm -f petclinic-staging || true
                docker run -d --name petclinic-staging --network miniproject-network -p 8081:8080 petclinic:staging
                sleep 30
                '''
            }
        }
        stage('ZAP Scan') {
            steps {
                sh '''
                docker exec zap zap-baseline.py -t http://petclinic-staging:8080 \
                    -r zap_report.html -x zap_report.xml || true
                '''
            }
            post {
                always {
                    publishHTML(target: [
                        reportDir: '.', reportFiles: 'zap_report.html',
                        reportName: 'ZAP Security Report', keepAll: true
                    ])
                }
            }
        }
    }
}