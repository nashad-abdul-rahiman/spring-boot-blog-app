// Uses Declarative syntax to run commands inside a container.
pipeline {
  agent none
  stages {
      stage('Build') {
      agent {
        kubernetes {

          yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: maven
    securityContext:
      privileged: true
    command:
     - cat
    tty: true
'''
         
          // Can also wrap individual steps:
          // container('shell') {
          //     sh 'hostname'
          // }
          defaultContainer 'shell'
        }
      }
      stages {
        stage('Checkout Source') {
          steps {
            git 'https://github.com/nashad-abdul-rahiman/spring-alien.git'
          }
        }
        
        stage('maven build') {
          steps {
            script {
              def mvn = tool 'maven default'
              sh "${mvn}/bin/mvn clean"
              sh "${mvn}/bin/mvn package"
            }
          }
        }
        
        stage('SonarQube Analysis') {
          steps {
            script {
              def mvn = tool 'maven default'
              withSonarQubeEnv() {
                  sh "${mvn}/bin/mvn sonar:sonar"
              }
              stash includes: '**/target/*.jar', name: 'app' 
              stash includes: 'Dockerfile', name: 'dockerfile' 
            }
          } 

        }
      }

      }  
      
      stage('Push to Repo') {
      agent {
        kubernetes {

          yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: docker
    image: docker:19.03.1-dind
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
'''
         
          // Can also wrap individual steps:
          // container('shell') {
          //     sh 'hostname'
          // }
          defaultContainer 'docker'
        }
      }
      stages {
        stage("Build image") {
          steps {
            unstash 'app'
            unstash 'dockerfile'
            sh 'docker build -t nashadabdulrahiman/spring-app:${BUILD_ID} .'
            sh 'docker tag nashadabdulrahiman/spring-app:${BUILD_ID} nashadabdulrahiman/spring-app:latest'
          }
        }
        stage("Push image") {
          steps {
            withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
              sh 'docker login -u $USERNAME -p $PASSWORD'
            }
            sh 'docker push nashadabdulrahiman/spring-app:${BUILD_ID}'
            sh 'docker push nashadabdulrahiman/spring-app:latest'
          }
        }          
      }
          
      }

   }
}
