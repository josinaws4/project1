pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(installationName: 'sonarqube') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: \${qg.status}"
                        }
                    }    
                }
            }
        }        
        stage("docker build & docker push to Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_login', variable: 'nexus_repo_password')]) {
                        sh '''
                            docker build -t 43.204.228.104:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_repo_password 43.204.228.104:8083
                            docker push  43.204.228.104:8083/springapp:${VERSION}
                            docker rmi 43.204.228.104:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        }
    }
}
