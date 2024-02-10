pipeline {
    agent { label 'Jenkins-Slave' }

    tools {
        maven "Maven"
    }

    stages {
	stage('SonarQube Analysis') {
            steps {
                script {
                    def mvn = tool 'maven';
                    withSonarQubeEnv('sonarqube') {
                        sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=greetapp_pipeline -f pom.xml"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        stage('Build') {
            steps {
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }

        stage('Test') {
		    steps {
			    script {
				      def mvn = tool 'maven';
                                      sh "${mvn}/bin/mvn -Dmaven.test.failure.ignore=true test"
                      }
                 }
				 }
		stage('Result') {
		    steps {
			    script {
				      def mvn = tool 'maven';
                                      junit(testResults: 'target/surefire-reports/*.xml', skipMarkingBuildUnstable: true, allowEmptyResults : true)
                      }
                 }
				 }
		stage('Deploy') {
                 steps {
                       script {
 
                             sh 'rsync -avz $WORKSPACE/target/GreetingApp-0.0.1-SNAPSHOT.jar jenkins@10.0.1.128:/tmp/'
                             sh 'ssh jenkins@10.0.2.58 "sudo -u greetings /usr/local/bin/deploy_greeting.sh"'
                              }
                        }
                     }
    }
}
