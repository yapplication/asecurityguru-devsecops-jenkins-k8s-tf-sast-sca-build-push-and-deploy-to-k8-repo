pipeline {
  agent any
  tools { 
        maven 'Maven_3_5_2'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=ysonarcloud -Dsonar.organization=ysonarcloud -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=55f9800b4a6a48584a4bbb13112662ece950e65a'
			}
    }

	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }

	stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("ydocker")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://108415871555.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:yaws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
	stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n ydevsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=ydevsecops')
		}
	      }
   	}

  }

  stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
        steps {
          withKubeConfig([credentialsId: 'kubelogin']) {
            sh('zap.sh -cmd -quickurl http://$(kubectl get services/ybuggy --namespace=ydevsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
            archiveArtifacts artifacts: 'zap_report.html'
          }
	     }
      }
}
