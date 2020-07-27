pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}
		
		stage('Build docker image') {
		    steps {
			    script{	
				    docker.withRegistry('https://registry.hub.docker.com','docker') {
					customImage = docker.build("ghadaj/mydockerwebapp")
			   		 }
				}
		    	}
		}
		stage('Push docker image') {
		    steps {
			    script{	
				    docker.withRegistry('https://registry.hub.docker.com','docker') {
					customImage.push()
					    }
				}
		    	}
		}
	/*	stage('Set current kubectl context') {
		    steps {
			    withAWS(region:'us-west-2', credentials:'aws'){
				    
				    sh 'kubectl config use-context arn:aws:eks:us-west-2:433927923947:cluster/windows-prod'
					}
				}
			} */
		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Wait user approve') {
            steps {
                input "Ready to redirect traffic to green?"
         	   }
      		  }

		stage('Create the service in the cluster, redirect to green') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
