pipeline {
  agent any
  tools { 
        maven 'maven_3_5_2'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=classapp -Dsonar.organization=classapp -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=5b960fff3f51281ed0b32fdb92655426773d5f32'
			}
    }
  
	stage('RunSCAAnalysisUsingSnyk') {
            steps {		
				withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
					sh 'mvn snyk:test -fn'
				}
			}
    }	


// building docker image
stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("adegokeimage")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
			
                    docker.withRegistry("https://820254520822.dkr.ecr.us-east-1.amazonaws.com", "ecr:us-east-1:aws-credentials") 
			{
                    app.push("latest")
                    }
                }
            }
    	}
  stage('Kubernetes deployment app'){
    steps {

      withKubeConfig([credentialsId: 'kubelogin']){
        sh('kubectl delete --all in devsecops')
        sh('kubectl apply -f deployment.yaml --nameapace=devsecops')
      }
    }
  }


  }
}
