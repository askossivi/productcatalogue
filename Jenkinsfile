currentBuild.displayName = "Java_App_Demo # "+currentBuild.number

   def getDockerTag(){
        def tag = sh script: 'git rev-parse HEAD', returnStdout: true
        return tag
        }
        

pipeline{
        agent {
		docker {
          		image 'maven'
          		args '-v $HOME/.m2:/root/.m2'
          	}
            } 
        environment{
	    Docker_tag = getDockerTag()
        }

// # SonaQube Quality Gate:        
        stages{
              stage('SonaQube Quality Gate Statuc Check'){
                  steps{
                      script{
                      withSonarQubeEnv('sonarserver') { 
                      sh "mvn sonar:sonar"
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

// # Build Maven Jar files
//  #-----------  Productcatalogue   ----------------        
              stage('Build Productcatalogue Maven App'){
              steps{
                  script{
		   sh "mvn clean install"
                       }
                    }
                 }

// //  #-----------  Shopfront   ---------------- 
//               stage('Build Shopfront Maven App'){
//               steps{
//                   script{
// 		   sh "mvn clean install  -f shopfront/pom.xml"
//                        }
//                     }
//                  }

// //  #-----------  Stockmanager   ---------------- 
//               stage('Build Stockmanager Maven App'){
//               steps{
//                   script{
// 		   sh "mvn clean install  -f stockmanager/pom.xml"
//                        }
//                     }
//                  }

// # Build Docker Images
// 1#--------------- Productcatalogue:
              stage('Build Productcatalogue Image'){
              steps{
                  script{
		   sh 'docker build . -t devtraining/productcatalogue:${BUILD_NUMBER}'
                       }
                    }
                 }
		 
// //  2#--------------- Shopfront:
//               stage('Build Shopfront Image'){
//               steps{
//                   script{
// 		   sh 'docker build . -t devtraining/shopfront:${BUILD_NUMBER}'
//                        }
//                     }
//                  }



// // #--------------- Stockmanager:

//               stage('Build Stockmanager Image'){
//               steps{
//                   script{
// 		   sh 'docker build . -t devtraining/stockmanager:${BUILD_NUMBER}'
//                        }
//                     }
//                  }
                 
// # Tags Push Docker Images	
	 
//  1#--------------- Productcatalogue:		
              stage('Push Productcatalogue Image'){
              steps{
                  script{
		   docker.withRegistry("https://index.docker.io/v1/", "Docker_Hub" ) {
                   sh 'docker push devtraining/productcatalogue:${BUILD_NUMBER}'
			}
                       }
                    }
                 }

// //  2#--------------- Shopfront:		
//               stage('Push Shopfront Image'){
//               steps{
//                   script{
// 		   docker.withRegistry("https://index.docker.io/v1/", "Docker_Hub" ) {
//                    sh 'docker push devtraining/shopfront:${BUILD_NUMBER}'
// 			}
//                        }
//                     }
//                  }

// //  3#--------------- Stockmanager:		
//               stage('Push Stockmanager Image'){
//               steps{
//                   script{
// 		   docker.withRegistry("https://index.docker.io/v1/", "Docker_Hub" ) {
//                    sh 'docker push devtraining/stockmanager:${BUILD_NUMBER}'
// 			}
//                        }
//                     }
//                  }


// # Deploy the aplication Using ARGOCD


// #-----------------
		stage('k8s Java Deployment'){
		steps{
		    script{
		      echo "triggering kubernetes-java-deployment job"
		    build job: 'kubernetes-java-deployment', parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
		    }
		}
	     }
          }
}






