pipeline {
	agent any
	stages {
        stage('Checking Requiremnts') {
              steps {          
                   sh 'docker -v'
                   sh 'eksctl version'
                   sh 'aws --version'
                  
              }
         }
		stage('Linting Html & Docker Files') {
			steps {
				sh 'tidy -q -e *.html'
				sh '/bin/hadolint --ignore DL3006 Dockerfile'
			}
		}
		
		stage('Build Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t adelm535/capstone .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push adelm535/capstone
					'''
				}
			}
		}

		stage('Set current kubectl context') {
						steps {
							withAWS(region:'us-east-2', credentials:'Aws') {
								sh 'kubectl config use-context arn:aws:eks:us-east-2:526842143734:cluster/capstone'
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-east-2', credentials:'Aws') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'us-east-2', credentials:'Aws') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-east-2', credentials:'Aws') {
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
				withAWS(region:'us-east-2', credentials:'Aws') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}
