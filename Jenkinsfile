// This is working example of Jenkinsfile for building and deploying Docker image to Kubernetes using Kaniko

def podTemplate = """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jenkins-slave
    image: jenkinsci/slave
    command:
    - sleep
    args:
    - infinity
  - name: kubectl
    image: alpine:latest
    command:
      - /bin/sh
      - -c
      - |
        apk add --no-cache curl
        curl -LO "https://s3.us-west-2.amazonaws.com/amazon-eks/1.25.6/2023-01-30/bin/linux/amd64/kubectl" 
        chmod +x kubectl
        mv kubectl /usr/local/bin/
        kubectl version --client
    tty: true
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
    - name: kaniko-secret
      secret:
        secretName: regcred
        items:
          - key: .dockerconfigjson
            path: config.json
"""

pipeline {

    agent {
        kubernetes {
            yaml podTemplate
            instanceCap 3
            defaultContainer 'jenkins-slave'
        }
    }

    stages {
        stage('Deploy') {
            steps {
                container('jenkins-slave') {
                    sh "echo hello world"
                }
            }
        }

        stage('Kaniko Build & Push Image') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                            /kaniko/executor --dockerfile `pwd`/Dockerfile \
                                             --context `pwd` \
                                             --destination=indamutsa/myweb:${BUILD_NUMBER}
                        '''
                    }
                }
            }
        }

        stage('Deploy App to Kubernetes') {
            steps {
                container('kubectl') {
                    withCredentials([file(credentialsId: 'mykubeconfig', variable: 'KUBECONFIG')]) {
                        sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
                        sh 'kubectl apply -f myweb.yaml'
                    }
                }
            }
        }
    }
}
