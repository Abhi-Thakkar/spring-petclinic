import hudson.model.*;
import hudson.FilePath;
pipeline{
    agent any
    stages{

        stage('Build'){
            steps {
                parallel(
                    "DB": {
                        sh './gradlew changelogSyncSQL --no-daemon'
                    },
                    "JAR": {
                        sh './gradlew bootJar --no-daemon'
                    }
                )
            }
        }

        stage('Unit Test') {
            steps{
                sh './gradlew test --no-daemon'
            }
            post {
                always {
                    junit(
                        allowEmptyResults: true,
                        testResults: '*/build/test-results/test/TEST-*.xml'
                    )
                }
            }
        }

        stage('Runtime Test') {
            steps{
                catchError {
                    timeout(time: 1, unit: 'MINUTES') {
                        sh 'SERVER_PORT=8888 MYSQL_URL=jdbc:mysql://mysql:3306/petclinic java -jar app/build/libs/app.jar'
                        sh 'curl localhost:8888/actuator/health'
                    }
                }
            }
        }

        stage('Publish') {
            steps{
                sh './gradlew jar publish --no-daemon'
            }
        }

        stage('Sonar') {
            steps{
                sh './gradlew sonar -Dorg.gradle.java.home=/usr/lib/jvm/java-11-openjdk-amd64 -x compileJava -x compileTestJava --no-daemon'
            }
        }
    }
}
