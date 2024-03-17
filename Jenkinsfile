/*pipeline {
    agent { label 'sa-javaslave' }

    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "slave_maven"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                echo 'Checkout Src from github repo'
		git 'https://github.com/LoksaiETA/Java-mvn-app2.git'
            }
        }
        stage('Maven Build') {
            steps {
                echo 'Perform Maven Build'
                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage('Deploy to QA Server') {
            steps {
		script {
		sshPublisher(publishers: [sshPublisherDesc(configName: 'QA_Server', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: '', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '.', remoteDirectorySDF: false, removePrefix: 'target/', sourceFiles: 'target/mvn-hello-world.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
		}
        }
	}
    }
}
*/

pipeline {
    agent any
    
    tools{
        jdk'jdk8'
        maven'maven3'
    }
    
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('GIT') {
            steps {
                git 'https://github.com/venkatesh9691/Java-mvn-app2.git'
            }
        }
        stage("Maven Compile"){
            steps{
                sh'mvn compile'
            }
        }
        stage("Maven Testing"){
            steps{
                sh'mvn test'
            }
        }
        stage("OWASP Dependency Checking"){
            steps{
                dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage("file system scanning"){
            steps{
	            sh'trivy fs --format table -o trivy-fs-report.html .'
            }
        }
        stage("Maven Build"){
            steps{
                sh'mvn clean package'
            }
        }
        stage("Nexus Artifact Uploader"){
            steps{
                nexusArtifactUploader artifacts: [
                    [
                        artifactId: 'mvn-hello-world',
                        classifier: '',
                        file: 'target/mvn-hello-world.war',
                        type: 'war'
                    ]
                ],
                credentialsId: 'Nexus',
                groupId: 'com.dev3l.hello_world', 
                nexusUrl: '16.170.141.202:8081',
                nexusVersion: 'nexus3',
                protocol: 'http',
                repository: 'venkatesh', 
                version: '1.0-SNAPSHOT'
            }
        }
        
        stage("Sonarqube analysis"){
            steps{
                withSonarQubeEnv('sonar-scanner') {
                    sh " mvn clean verify sonar:sonar -Dsonar.token=bb33547bf168d918256d89936b62299f4a03404b"
                }
            }
        }
        
        stage('Build Docker Image'){
            steps{
                sh'docker stop s4'
                sh 'docker rm s4'
                sh 'docker build -t venkatesh9691/venkatesh-projects-new .'
                sh 'docker build -t tomcat:${BUILD_NUMBER} .'
                sh 'docker run -itd --name s4 -p 3800:8080 tomcat:${BUILD_NUMBER}'
            }
        }
        stage("Docker image scaning"){
            steps{
                sh'trivy image --format table -o trivy-fs-report.html venkatesh9691/venkatesh-projects-new'
            }
        }
        
        stage("Pushing Docker image to HUB"){
            steps{
                withCredentials([string(credentialsId: 'DOCKER_HUB_CREDENTIALS', variable: 'DOCKER_HUB_CREDENTIALS')]) {
                    sh "docker login -u venkatesh9691 -p ${DOCKER_HUB_CREDENTIALS}"
                }
                sh 'docker push venkatesh9691/venkatesh-projects-new'
            }
        }
    }
}
