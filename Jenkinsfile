currentBuild.displayName = "Java_Demo_Webapp # "+currentBuild.number

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
                      sh "mvn sonar:sonar -Dsonar.java.binaries=target/classes"
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


// # Build Docker Images
// 1#--------------- Productcatalogue:
              stage('Build Productcatalogue Image'){
              steps{
                  script{
		   sh 'docker build . -t devtraining/productcatalogue:${BUILD_NUMBER}'
                       }
                    }
                 }
		 
                 
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


// # Build Shopfront Job


// #-----------------
		stage('Build Shopfront Job'){
		steps{
		    script{
		      echo "triggering Shopfront Build Job"
		    build job: 'shopfront-build'//, parameters: [string(name: 'DOCKERTAG', value: env.BUILD_NUMBER)]
		    }
		}
	     }
          }
}







