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
                            docker build -t 54.185.115.104/springapp:${VERSION} .
                            docker login -u admin -p admin 54.185.115.104
                            docker push 54.185.115.104/springapp:${VERSION}
                            docker rmi 54.185.115.104/springapp:${VERSION}
                           '''
                    }
                 }
             }
         }
    }
}           
