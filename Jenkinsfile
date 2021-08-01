// Define paramerter
def scmVars
// Uses Declarative syntax to run commands inside a container.

pipeline {
    agent {

        // user kubernetes as dynamic slave jenkins
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: docker
      image: docker:20.10.3-dind
      command:
      - dockerd
      tty: true
      securityContext:
        privileged: true
    - name: helm
      image: lachlanevenson/k8s-helm:v3.6.0
      command:
      - cat
      tty: true
    - name: java-node
      image: timbru31/java-node:11-alpine-jre-14
      command:
      - cat
      tty: true
"""
    } // End kubernetes 
  } // End agent
  
  environment {
    ENV_NAME = "${BRANCH_NAME == "master" ? "uat" : "${BRANCH_NAME}"}"
    SCANNER_HOME = tool 'sonarqube-scanner'
    PROJECT_KEY = "bookinfo-ratings-key"
    PROJECT_NAME = "bookinfo-ratings"
  }

//Start pipeline
  stages {
      // Clones the repository 
    stage('Clone source code') {
      steps {
        container('jnlp'){
          script {
            scmVars = git branch: '${BRANCH_NAME}',
                          credentialsId: 'bookinfo-git-deploy-key',
                          url: 'git@github.com:mercurial963/bookinfo-ratings.git'
                  }// end script
              }// end container
          }// end steps
      }// end stage
    stage('Sonarqube Scanner') {
      steps {
        container('java-node'){
          script {
            withSonarQubeEnv('sonarqube-jenkins'){

              sh """${SCANNER_HOME}/bin/sonar-scanner \
              -D sonar.projectKey=${PROJECT_KEY} \
              -D sonar.projectName=${PROJECT_NAME} \
              -D sonar.projectVersion=${BRANCH_NAME}-${BUILD_NUMBER} \
              -D sonar.source=./src
              """
            } // end withSonarQubeEnv

            timeout(time:1, unit: 'MINUTE') {//Just in case something goes wrong,
              def qg = waitForQualityGate() //Reuse TaskID
              if (qg.status != 'OK'){
                error = "Pipeline aborted due to quality gate failure: ${qg.status}"
              }
            } // end timeout
                  }// end script
              }// end container
          }// end steps
      }// end stage 

      // Build image Dockerfile and push 
    stage('Build and Push') {

      steps {
        container('docker'){
          script {
            docker.withRegistry('https://ghcr.io', 'registry-bookinfo'){ //registry-bookinfo is user with token
                          // build and push
              docker.build('ghcr.io/mercurial963/bookinfo-ratings:${ENV_NAME}').push()
              }// end docker.withRegistry

                  }// end script
              }// end container
          }// end steps
      }// end stage

      // Deploy
    stage('Deploy ratings with Helm Chart') {
      steps {
        container('helm'){
          script {
            // withKubeConfig([credentialsId: 'config']){ //add kubeconfig to secret file
              sh "helm upgrade --install -f helm-values/values-bookinfo-${ENV_NAME}-ratings.yaml --wait --set extraEnv.COMMIT_ID=${scmVars.GIT_COMMIT} --namespace ${ENV_NAME} bookinfo-${ENV_NAME}-ratings helm/"
              // }// withCredentials

                  }// end script
              }// end container
          }// end steps
      }// end stage          
  }// end stages
  }



