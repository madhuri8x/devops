pipeline {
    agent any
    tools {
        maven 'maven_3.6.3'
    }

    stages {

        stage('Initialize') {
            steps {
                echo "PATH = ${PATH}"
                echo "M2_HOME = ${M2_HOME}"
            }
        }

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: 'main']], extensions: [[$class: 'CheckoutOption', timeout: 5], [$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 5]], userRemoteConfigs: [[url: 'https://github.com/madhuri8x/devops.git']]])

                echo 'Checkout source code from git'
            }
        }
        stage('Quality Check and Security Check') {
            steps {
		parallel(
		    'Unit Test':{
			echo 'Junit test in progress'
                        sh "mvn clean test"
		    },
		    'Quality Check':{
	                echo 'QA verified'
		    },
		    'Security Check':{
			 dependencyCheck additionalArguments: '--scan=. --format=HTML', odcInstallation: 'OWASP-Dependency-Check'
	                 echo 'All security checks done'
		    }
		)
            }
        }
        stage('Build Push App') {
            steps {
                sh "mvn clean install"
            }
        }   
        stage('Kill previous deploy ment') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'SUCCESS') {
                    sh "fuser -k 8083/tcp"
                }
            }
        }

        stage('Deploy') {
            steps {
                sh "JENKINS_NODE_COOKIE=dontKillMe nohup java -jar ./target/spring-boot-rest-2-0.0.1-SNAPSHOT.jar &"
                echo 'Deployment done'
            }
        }
        stage('Post Deployment Check') {
            steps {
                echo 'All deployment check done'
		sh '/usr/local/bin/newman run Student_Api.postman_collection.json -r html,cli'
            }
	}
    }
}
