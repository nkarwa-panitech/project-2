pipeline {
    agent {label 'node01'}
    options { skipDefaultCheckout() }
    environment{
	    Docker_tag = "${env.BUILD_NUMBER}"
        AWS_DEFAULT_REGION="us-east-1"
        }
    tools { 
        maven '3.9.6' 
    }
    stages {
        stage('Checkout git') {
            steps {
               git branch: 'main', url: 'https://github.com/nkarwa-panitech/project-2.git'
            }
        }
        
        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn package' 
            }
            post {
               success {
                    junit 'target/surefire-reports/**/*.xml'
                }   
            }
        }

        stage("ArchiveArtifacts"){

            steps{
               archiveArtifacts artifacts: 'target/myapp-1.0.jar', followSymlinks: false
            }
	}
        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('sonar') {
                        sh "mvn sonar:sonar"
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
    }
}
