pipeline {
    agent any
    environment {
	SONAR_HOST_URL='=http://localhost:9000'
	SONAR_LOGIN='sqp_4657f17bac0629bf48e7f49f36d43ef7e97aad9b'
    
        // This can be nexus3 or nexus2
        NEXUS_VERSION = 'nexus3'
        // This can be http or https
        NEXUS_PROTOCOL = 'http'
        // Where your Nexus is running. In my case:
        NEXUS_URL = 'localhost:8081'
        // Repository where we will upload the artifact
        NEXUS_REPOSITORY = 'maven-snapshots'
        // Jenkins credential id to authenticate to Nexus OSS
        NEXUS_CREDENTIAL_ID = 'nexus-user-credentials'

    }
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
		when {
				branch 'sonar/*'
		}
		steps {
			sh 'mvn clean verify sonar:sonar \
				  -Dsonar.projectKey=mon-appli \
				  -Dsonar.host.url=$SONAR_HOST_URL \
				  -Dsonar.login=$SONAR_LOGIN'
		}
	}
	stage('Deploy Artifact To Nexus') {
            steps {
                script {
                    echo "ls"
                    echo "ls target"
                    pom = readMavenPom file: 'pom.xml'
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    artifactPath = filesByGlob[0].path
                    artifactExists = fileExists artifactPath
                    if (artifactExists) {
                        nexusArtifactUploader(
                        nexusVersion: NEXUS_VERSION,
                        protocol: NEXUS_PROTOCOL,
                        nexusUrl: NEXUS_URL,
                        groupId: pom.groupId,
                        version: pom.version,
                        repository: NEXUS_REPOSITORY,
                        credentialsId: NEXUS_CREDENTIAL_ID,
                        artifacts: [
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: artifactPath,
                            type: pom.packaging
                            ],
                            [artifactId: pom.artifactId,
                            classifier: '',
                            file: 'pom.xml',
                            type: 'pom'
                            ]
                        ]
                        )
                    } else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
	}
    }
}
