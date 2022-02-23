pipeline{
    agent any
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
                     withCredentials([string(credentialsId: 'nexus-password', variable: 'nexus-cred')]) {
                        
                            docker build -t 35.88.209.60:8083/springapp:${VERSION} .
                            docker login -u admin -p $nexus-credentials 35.88.209.60:8083
                            docker push 35.88.209.60:8083/springapp:${VERSION}
                            docker rmi 35.88.209.60:8083/springapp:${VERSION}

                           '''
                    }
                 }
             }
         }
    }
}           
