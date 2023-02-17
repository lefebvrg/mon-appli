pipeline {
    agent any
    stages {
	stage ('SCM') {
		steps {
			checkout scm
		}
	}
	stage ('Compile') {
		agent {	
       			docker {
       				image 'maven:3.6.0-jdk-8-alpine'
				reuseNode true
			}
		}
       		steps {
       			sh 'mvn -B -DskipTests clean package'
    		}
	}
	stage ('Tests unitaires') {
		steps {
			sh 'mvn test'
		}
		post {
			always {
			    junit 'target/surefire-reports/*.xml'
			}
		}
	}
	stage ('CheckStyle') {
		steps {
		    sh 'mvn checkstyle:checkstyle'
		}
		post {
		   always {
			recordIssues enabledForFailure: true, tool: checkStyle()
		    }
		}
	}
	stage ('Code analyze') {
		steps {
	    		sh 'mvn clean verify sonar:sonar \
				  -Dsonar.projectKey=mon-appli \
				  -Dsonar.host.url=http://localhost:9000 \
				  -Dsonar.login=sqp_4657f17bac0629bf48e7f49f36d43ef7e97aad9b'
		}
	}
    }
}
