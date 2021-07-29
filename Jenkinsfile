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
    """
    } // End kubernetes
    
} // End agent
//Start pipeline
stages {
    // Clones the repository 
  stage('Clone source code') {
    steps {
      container('jnlp'){
        script {
          scmVars = git branch: 'dev',
                        credentialsId: 'bookinfo-git-deploy-key',
                        url: 'git@github.com:mercurial963/bookinfo-ratings.git'
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
            docker.build('ghcr.io/mercurial963/bookinfo-ratings:dev').push()
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
          // withKubeConfig([credentialsId: 'config',serverUrl: '172.17.0.2:6443']){ //add kubeconfig to secret file
            sh "helm upgrade --install -f helm-values/values-bookinfo-dev-ratings.yaml --wait --set extraEnv.COMMIT_ID=${scmVars.GIT_COMMIT} --namespace dev bookinfo-dev-ratings helm/"
            }// withCredentials

                // }// end script
            }// end container
        }// end steps
    }// end stage    
}// end stages
}
