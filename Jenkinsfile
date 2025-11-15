pipeline {
  agent any
  tools { 
        maven 'Maven'  
    }
   stages{
    stage('CompileandRunSonarAnalysis') {
            steps {	
		sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=obalola14_buggy_code -Dsonar.organization=obalola14 -Dsonar.host.url=https://sonarcloud.io -Dsonar.token=7295af736b02e64c5e4624e456b4c4444f0b611a'
			}
        } 
  }
}
