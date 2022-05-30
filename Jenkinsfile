pipeline{
    agent any
    stages{
        stage("sonar quality check"){
            agent{
                docker{
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(installationName: 'sonarqube') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube -Dsonar.host.url=http://65.0.21.139:9000 -Dsonar.login=1b05f2ee147d459c18d140b9f54ef6a6ac03f562'
                    }
                }
            }
        }
    }
}
