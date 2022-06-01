pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{

        stage("docker build & docker push to Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_login', variable: 'nexus_repo_password')]) {
                        sh '''
                            docker build -t 3.110.185.92:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus_repo_password 3.110.185.92:8083
                            docker push  3.110.185.92:8083/springapp:${VERSION}
                            docker rmi 3.110.185.92:8083/springapp:${VERSION}
                        '''
                    }
                }
            }
        } 
    }
}        
