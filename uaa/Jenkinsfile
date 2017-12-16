#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    docker.image('openjdk:8').inside('-u root -e MAVEN_OPTS="-Duser.home=./"') {
        stage('check java') {
            sh "java -version"
        }

        stage('clean') {
            sh "chmod +x mvnw"
            sh "./mvnw clean"
        }
        
        stage('install tools') {
			    
                sh "./mvnw com.github.eirslett:frontend-maven-plugin:install-node-and-yarn -DnodeVersion=v8.9.1 -DyarnVersion=v1.3.2"
            }

       stage('yarn install') {			    
                sh "./mvnw com.github.eirslett:frontend-maven-plugin:yarn"
            }

        stage('backend tests') {
            try {
                sh "./mvnw test"
            } catch(err) {
                throw err
            } finally {
                junit '**/target/surefire-reports/TEST-*.xml'
            }
        }

  	 stage('package') {
		 	 	sh "set NODE_TLS_REJECT_UNAUTHORIZED=0"
		 	 	sh "npm config set strict-ssl false"
		 	 	sh "rm -R node && rm -R node_modules"
                sh "./mvnw  -Pprod package -DskipTests"
          }

    }

    def dockerImage
    stage('build docker') {
        sh "cp -R src/main/docker target/"
        sh "cp target/*.war target/docker/"
        dockerImage = docker.build('docker-login/jhipstercassandra', 'target/docker')
    }

    stage('publish docker') {
        docker.withRegistry('192.168.99.100:5000', 'docker-login') {
            dockerImage.push 'latest'
        }
    }
}
