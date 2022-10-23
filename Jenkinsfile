pipeline{
    agent any
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