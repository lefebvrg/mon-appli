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
    }
}
