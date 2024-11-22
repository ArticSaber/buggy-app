pipeline {
  agent any
  tools { 
        maven 'maven'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=sampleapp-devsecops_sampleproject-devsecops -Dsonar.organization=sampleapp-devsecops -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=c1384e5de845c5471b5b72ce18a40df7eaee680c'
			}
        } 
	   
stage('RunSCAAnalysisUsingSnyk') {
    steps {
        script {
            sh 'chmod +x mvnw'
        }
        withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
            sh 'mvn snyk:test -X -fn'
        }
    }
} 
	   stage('Build') { 
            steps { 
               withDockerRegistry([credentialsId: "dockerlogin", url: ""]) {
                 script{
                 app =  docker.build("mydevsecopsproject")
                 }
               }
            }
    }

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://027423892923.dkr.ecr.us-east-2.amazonaws.com', 'ecr:us-east-2:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}
	   
        stage('Kubernetes Deployment of Vuln-App Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) {
		  sh('kubectl delete all --all -n devsecops')
		  sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
		}
	      }
   	}

  }
}
