pipeline {
  agent any
  tools { 
      maven 'Maven'  
    }
   stages{
      stage('CompileandRunSonarAnalysis') {
            steps {	
                sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=obalola14_buggy_code -Dsonar.organization=obalola14 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=7295af736b02e64c5e4624e456b4c4444f0b611a'
            //   withSonarQubeEnv('sonar') {
            //     sh 'mvn clean verify sonar:sonar'
            //   }
            // verify is the last stage of build lifecycle for mvn and it runs any checks to verify the project is valid and meets quality criteria. sonar:sonar runs the sonar analysis. -Dsonar.* are the parameters required for sonarcloud analysis. specific to your Maven project.
            }
  }


      // stage('RunSCAAnalysisUsingSnyk') {
      //       steps {		
      //                   withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
      //                         // sh 'mvn snyk:test -fn' 
      //                         sh 'mvn snyk:test -Dsnyk.test.jsonReport=true -Dsnyk.test.jsonReportOutputFile=snyk_report.json -fn'
      //                         archiveArtifacts artifacts: 'snyk_report.json', fingerprint: true
      //                         // -fn stops jenkins job from failing if snyk frinds vulnerabilities
      //                   }
      //             }
      //       }	


      stage('Snyk Scan') {
            steps {
                  withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
                        // 1️⃣ Run Snyk Maven plugin (uploads results to Snyk UI)
                        sh """
                        export SNYK_TOKEN=${SNYK_TOKEN}
                        mvn snyk:test -fn
                        """

                        // 2️⃣ Run Snyk CLI to produce local JSON report for Jenkins
                        sh """
                        export SNYK_TOKEN=${SNYK_TOKEN}
                        snyk test --file=pom.xml --json > snyk_report.json
                        """

                        // 3️⃣ Archive the JSON report in Jenkins
                        archiveArtifacts artifacts: 'snyk_report.json', fingerprint: true

                        // 4️⃣ (Optional) Convert to HTML for human-readable report
                        // snyk-to-html -i snyk_report.json -o snyk_report.html
                        // archiveArtifacts artifacts: 'snyk_report.html'
                  }
            }
      }



//steps below builds docker image and pushes to AWS ECR, different from using dokcer commands in shell script. Made possible with Jenkins Docker Pipleline plugin
      stage('Build') { 
            steps { 
                  withDockerRegistry([credentialsId: "dockerhub_login", url: ""]) {
                  script{
                  app =  docker.build("asg")
                  //docker.build command above is the same as docker build -t asg .                 
                  }
            }
      }
}

	stage('Push') {
            steps {
                script{
                    docker.withRegistry('https://306617143793.dkr.ecr.us-west-2.amazonaws.com/asg', 'ecr:us-west-2:aws-credentials') {
                    app.push("latest")
                    }
                }
            }
    	}

      stage('Kubernetes Deployment of ASG Bugg Web Application') {
	   steps {
	      withKubeConfig([credentialsId: 'kubelogin']) { //Kubernetes CLI Plugin
		sh ('kubectl delete all --all -n devsecops')
		sh ('kubectl apply -f deployment.yaml --namespace=devsecops')
//          can also do
//          withCredentials([file(credentialsId: 'kubeconfig-prod', variable: 'KCFG')]) {
//          sh '''
//          export KUBECONFIG=$KCFG
//          kubectl get nodes
//     '''
                  }
            }     
   	}


      stage ('wait_for_testing'){
	   steps {
		   sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'//waits for 3 mins to allow kubernetes and app to be fully deployed before DAST testing
	   	}
	   }
	   
	stage('RunDASTUsingZAP') {
          steps {
		    withKubeConfig([credentialsId: 'kubelogin']) {
				sh('zap.sh -cmd -quickurl http://$(kubectl get services/asgbuggy --namespace=devsecops -o json| jq -r ".status.loadBalancer.ingress[] | .hostname") -quickprogress -quickout ${WORKSPACE}/zap_report.html')
				archiveArtifacts artifacts: 'zap_report.html' , fingerprint: true
		    }
	     }
       } 
      // stage('ZAP Scan') {
      //       steps {
      //             script {
      //                   withKubeConfig([credentialsId: 'kubelogin']) {
      //                   // Resolve URL on the Jenkins node (where kubectl exists)
      //                   def targetUrl = sh(
      //                   script: "kubectl get svc asgbuggy -n devsecops -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'",
      //                   returnStdout: true
      //                   ).trim()

      //                   echo "Target URL: http://${targetUrl}"

      //                   // Use Jenkins docker credentials for authenticated pulls
      //                   withDockerRegistry([credentialsId: 'dockerhub_login', url: 'https://index.docker.io/v1/']) {

      //               // Run ZAP inside the Docker container

      //                         docker.image('owasp/zap2docker-stable').inside('-u root:root') {
      //                               sh """
      //                                     zap.sh -cmd \
      //                                           -quickurl http://${targetUrl} \
      //                                           -quickprogress \
      //                                           -quickout zap_report.html
      //                         """
      //                   }
      //             }
      //       }
      //       }
      // }

     // }
	      
}
}

