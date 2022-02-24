pipeline{
    agent any
    environment{
        VERSION  = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar quality check") {
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-password ') {
                           sh 'chmod +x gradlew'
                           sh './gradlew sonarqube'
                        }
                      timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"   
                        }
                    }

                }

            }
        }
        stage ("docker build & docker push"){
             steps{
                 script{
                     withCredentials([string(credentialsId: 'nexus', variable: 'nexus-pwd')]) {
                        sh '''
                            docker build -t 54.185.115.104:8083/springapp:${VERSION} .
                            docker login -u admin -p admin 54.185.115.104:8083
                            docker push 54.185.115.104:8083/springapp:${VERSION}
                            docker rmi 54.185.115.104:8083/springapp:${VERSION}
                           '''
                    }
                 }
             }
         }
        stage ("Push the help chars to nexus"){
             steps{
                 script{
                     withCredentials([string(credentialsId: 'nexus', variable: 'nexus-pwd')]) {
                         dir('kubernetes/') {
                        sh '''
                             helmversion=$( helm show chart myapp | grep version | cut -d: -f 2 | tr -d ' ')
                             tar -czvf  myapp-${helmversion}.tgz myapp/
                             curl -u admin:admin //http://54.185.115.104:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                            '''
                         }   
                    }
                 }
             }
         }
    }

}           
