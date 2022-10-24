pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("sonar_quality_check"){
            // agent {
            //     // docker {
            //     //     image 'openjdk:11'
            //     // }
            // }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'jenkins_token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube --stacktrace'
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
        stage("docker build & docker push"){
            steps{
                script{

                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_pass')]) {
       
                        sh '''
                            docker build -t 174.129.132.61:8083/springapp:${VERSION} .
                            docker login -u admin -p $docker_pass 174.129.132.61:8083
                            docker push 174.129.132.61:8083/springapp:${VERSION}
                            docker rmi 174.129.132.61:8083/springapp:${VERSION}
                        '''
                      }
                }
            }
        }
    }
    post{
        always{
            echo "SUCCESS"
        }
        
    }
}