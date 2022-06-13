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
                            docker build -t nexus.josin.ml:9999/springapp:${VERSION} .
                            docker login -u admin -p $nexus_repo_password nexus.josin.ml:9999
                            docker push  nexus.josin.ml:9999/springapp:${VERSION}
                            docker rmi nexus.josin.ml:9999/springapp:${VERSION}
                        '''
                    }
                }
            }
        }
        stage("Identifying misconfig using datree in Helm chart"){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=c6c0e32c-e995-4c15-b120-2e64ae1bc220']) {
                            sh 'helm datree test myapp/'
                        }            
                    }
                }
            }
        }
        stage("Pushing the Helm Chart to Nexus"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'nexus_login', variable: 'nexus_repo_password')]) {
                        dir('kubernetes/') {
                            sh '''
                                helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                                tar -czvf  myapp-${helmversion}.tgz myapp/
                                curl -u admin:$nexus_repo_password http://nexus.josin.ml:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v

                            '''
                        }
                    }
                }
            }
        }
        stage('manual approval'){
            steps{
                script{
                    timeout(10) {
                        mail to: "crowne2007@gmail.com",
                        subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}",
                        body: "Build Number: ${env.BUILD_NUMBER} \nGo to build url and approve the deployment request. \nBuild URL: ${env.BUILD_URL}"                        
                        input(message: "Deploy ${env.JOB_NAME} job ?", ok: 'Deploy')
                    }
                }
            }
        }
        stage('Deploying Application on K8 Cluster') {
            steps {
                script{
                    withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        dir('kubernetes/') {
                            sh 'helm upgrade --install --set image.repository="nexus.josin.ml:9999/springapp" --set image.tag="${VERSION}" myjavaapp myapp/'
                        }
                    }
                }
            }
        }

        stage('verifying app deployment'){
            steps{
                script{
                     withCredentials([kubeconfigFile(credentialsId: 'kubernetes-config', variable: 'KUBECONFIG')]) {
                        timeout(10) {
                            sh 'kubectl run curl --image=curlimages/curl -i --rm --restart=Never -- curl myjavaapp-myapp:8080'
                        }
                     }
                }
            }
        }
    }
    post{
        always{
            mail to: "crowne2007@gmail.com",
            subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}",
            body: "Build Number: ${env.BUILD_NUMBER} \nBuild URL: ${env.BUILD_URL}"
        }
    }
}
