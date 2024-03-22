pipeline {
    agent { label 'jenkins-agent' }
    tools {
        jdk 'java17'
        maven 'maven3'
    }
	environment {
	    APP_NAME = "register-app-pipeline"
            RELEASE = "1.0.0"
            DOCKER_USER = "johnsonjo100"
            DOCKER_PASS = 'dockerhub'
            IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
            IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
    }
    stages{
        stage("Cleanup Workspace"){
                steps {
                cleanWs()
                }
        }

        stage("Checkout from SCM"){
                steps {
                    git branch: 'main', credentialsId: 'github', url: 'https://github.com/johnson-keyshell/git-jenkins-pipeline.git'
                }
        }

        stage("Build Application"){
            steps {
                sh "mvn clean package"
            }

       }

       stage("Test Application"){
           steps {
                 sh "mvn test"
           }
       }
       stage("SonarQube Analysis"){
           steps {
	           script {
		        withSonarQubeEnv(credentialsId: 'jenkins-sonarQube-token') { 
                        sh "mvn sonar:sonar"
		        }
	           }	
           }
       }
    
       stage("Quality Gate"){
           steps {
               script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarQube-token'
                }	
            }
        }
	 stage('Install Tomcat 9') {
            steps {
                sh 'sudo apt install -y tomcat9 tomcat9-admin tomcat9-examples'
            }
        }
        
        stage('Set Tomcat directory ownership') {
            steps {
                sh 'sudo chown -R tomcat:tomcat /var/lib/tomcat9/'
            }
        }
        
        stage('Make Tomcat script executable') {
            steps {
                sh 'sudo chmod +x /var/lib/tomcat9'
            }
        }
        
        stage('deploy the application') {
            steps {
                dir('/home/ubuntu/workspace/register-app-ci')
                sh 'sudo cp /home/ubuntu/workspace/register-app-ci/webapp/target/webapp.war /var/lib/tomcat9/webapps/'
            }
        }
        
        stage('Start Tomcat') {
            steps {
                sh 'sudo systemctl start tomcat9'
            }
        }
    }
}
    
        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
		    }
		}
	    }
	}
        stage("Trivy Scan") {
           steps {
               script {
	            sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image johnsonjo100/register-app-pipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
               }
           }
       }

       stage ('Cleanup Artifacts') {
           steps {
               script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
               }
          }
       }
}

		

		 
	     
	    

    


