def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }

pipeline {
    agent {label 'node01'}
    options { skipDefaultCheckout() }
    environment{
	    Docker_tag = getDockerTag()
        AWS_DEFAULT_REGION="us-east-1"
        }
    tools { 
        maven '3.9.4' 
    }
    stages {
	// stage('CleanWorkspace') {
 //            steps {
 //                cleanWs()
 //            }
	// }
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
        // stage('SonarQube Analysis'){
        //     steps{
        //         withSonarQubeEnv('sonar') {
        //                 sh "mvn sonar:sonar"
        //         }
        //     }
        // }
        // stage("Quality Gate") {
        //     steps {
        //       timeout(time: 1, unit: 'HOURS') {
        //         waitForQualityGate abortPipeline: true
        //       }
        //     }
        // }
        
        stage('Docker Build & Push') {
            steps {
      	        sh 'docker build -t nkarwapanitech/sprint-boot-app:$Docker_tag .'
                withCredentials([string(credentialsId: 'dockerhublogin', variable: 'docker_password')]) {		    
				  sh 'docker login -u nkarwapanitech -p $docker_password'
				  sh 'docker push nkarwapanitech/sprint-boot-app:$Docker_tag'
			}
            }
        }
        // stage('Image Scan') {
        //     steps {
      	 //        sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html nkarwapanitech/sprint-boot-app:$Docker_tag '
        //     }
        // }
        // stage('Upload Scan report to AWS S3') {
        //     //   steps {
        //     //       sh 'aws s3 cp report.html s3://panitech-devsecops-project/'
        //     //   }
        //     steps {
        //       withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'myaws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
        //         sh '''
        //           aws s3 cp report.html s3://panitech-devsecops-project/
        //            '''
        // }
        //  }
        // }
        
       //  stage("Approval"){

       //  steps{

       //      timeout(time: 15, unit: 'MINUTES'){ 
	      //          input message: 'Do you approve deployment for production?' , ok: 'Yes'}

       //  }
       // }

        stage ('Prod pre-request') {
            agent { label 'node01' }
			steps{
			 	script{
				    sh ''' ls -lrt
	                             final_tag=$(echo $Docker_tag | tr -d ' ')
				     echo ${final_tag}test
				     sed -i "s/docker_tag/$final_tag/g"  spring-boot-deployment.yaml
	                             cat spring-boot-deployment.yaml
				     '''
				}
			}
        }
        stage('Deploy to k8s') {
              steps {
                script{
                    kubernetesDeploy configs: 'spring-boot-deployment.yaml', kubeconfigId: 'kubeconfig'
                }
            }
        }
        
 
    }
    post{
        always{
            sendSlackNotifcation()
            }
        }
}
def sendSlackNotifcation()
{
    if ( currentBuild.currentResult == "SUCCESS" ) {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *SUCCESS*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL} \n"
        slackSend( channel: "#class-1", token: 'slack', color: 'good', message: "${buildSummary}")
    }
    else {
        buildSummary = "Job_name: ${env.JOB_NAME}\n Build_id: ${env.BUILD_ID} \n Status: *FAILURE*\n Build_url: ${BUILD_URL}\n Job_url: ${JOB_URL}\n  \n "
        slackSend( channel: "#class-1", token: 'slack', color : "danger", message: "${buildSummary}")
    }
}

    
