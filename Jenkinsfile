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

        stage("identifying misconfigs using datree in helm charts"){
            steps{
                script{
                    dir('kubernetes/myapp'){
                        withEnv(['DATREE_TOKEN=6cb6b8ac-e0e3-4bb1-a11b-5f0ff7b5def3']) {
                            
                        }
                       
                    }
                }
            }
        }

            stage("Pushing helm charts to nexus "){
                steps{
                    script{

                        withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_pass')]) {
                            dir("'kubernetes/myapp'"){
                                sh '''
                                helmversion=${helm show chart myapp | grep version |cut -d: -f 2 | tr -d ''}
                                tar -czf myapp-${helmversion}.tgz myapp/
                                curl -u admin:$docker_pass http://174.129.132.61:8081/repository/helm-hosted/ --upload-file myapp-${helmversion}.tgz -v
                                '''
                            }
                        }
                    }
                }
        }

    }
   
    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "maaroufi.oussama08@gmail.com";  
		}
	}
        
    
}