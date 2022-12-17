pipeline {
  agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: maven
            image: maven:alpine
            command:
            - cat
            tty: true
          - name: docker
            image: docker:latest
            command:
            - cat
            tty: true
            volumeMounts:
             - mountPath: /var/run/docker.sock
               name: docker-sock
          volumes:
          - name: docker-sock
            hostPath:
              path: /var/run/docker.sock    
        '''
    }
  }
  	environment {
		DOCKERHUB_CREDENTIALS=credentials('docker')
	}
  stages {
    // stage('Clone') {
    //   steps {
    //     container('maven') {
    //       git branch: 'main', changelog: false, poll: false, url: 'https://mohdsabir-cloudside@bitbucket.org/mohdsabir-cloudside/java-app.git'
    //     }
    //   }
    // }  
    stage('Build-Jar-file') {
      steps {
        container('maven') {
          sh 'mvn package'
        }
      }
    }
    stage('Build-Docker-Image') {
      steps {
        container('docker') {
          sh 'docker build -t dareyregistry/java-app:latest .'
        }
      }
    }

		stage('Docker Login') {

			steps {
        withCredentials([usernamePassword(credentialsId: 'docker', passwordVariable: 'Docker_registry_password', usernameVariable: 'Docker_registry_user')]) {
            // some block
            sh 'docker login -u $Docker_registry_user -p $Docker_registry_password'
        }
		}

    // stage('Login-Into-Docker') {
    //   steps {
    //     container('docker') {
    //       sh 'docker login -u dareyregistry -p Phartion001ng'
    //   }
    // }
    // }
     stage('Push-Images-Docker-to-DockerHub') {
      steps {
        container('docker') {
          sh 'docker push dareyregistry/java-app:latest'
      }
    }
     }
  }
    post {
      always {
        container('docker') {
          sh 'docker logout'
      }
      }
    }
   }
  }
