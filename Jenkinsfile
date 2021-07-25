// Uses Declarative syntax to run commands inside a container.
pipeline {
    agent {
        // user kubernetes as dynamic slave jenkins
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            spec:
            containers:
            - name: docker
                image: docker:20.10.7-dind
                command:
                - dockerd
                tty: true
                securityContext:
                privileged: true
'''
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
                container('jnlp'){
                    script {
                        docker.withRegistry('https://github.com/mercurial963', 'registry_bookinfo'){
                            // build and push
                            docker.build('ghcr.io/mercurial963/bookinfo-ratings:dev').push()
                        }// end docker.withRegistry
                    }// end script
                }// end container
            }// end steps
        }// end stage
    }// end stages
}
