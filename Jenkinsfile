// Define a pod template for Jenkins pipeline
def podTemplate = """
apiVersion: v1
kind: Pod
spec:
  containers:
  # Jenkins slave container
  - name: jenkins-slave
    image: jenkinsci/slave
    command:
    - sleep
    args:
    - infinity
  # Kubectl container for interacting with Kubernetes
  - name: kubectl
    image: alpine/k8s:1.24.12
    command:
      - /bin/sh
    tty: true
  # Kaniko container for building and pushing Docker images
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - /busybox/cat
    tty: true
    volumeMounts:
      - name: kaniko-secret
        mountPath: /kaniko/.docker
  volumes:
    # Mount the registry secret for Kaniko to authenticate
    - name: kaniko-secret
      secret:
        secretName: regcred
        items:
          - key: .dockerconfigjson
            path: config.json
"""

// Define the Jenkins pipeline
pipeline {
    agent {
        kubernetes {
            yaml podTemplate
            instanceCap 6
            defaultContainer 'jenkins-slave'
        }
    }

    stages {
        // Deploy stage (for demonstration purposes)
        stage('Deploy') {
            steps {
                container('jenkins-slave') {
                    sh "echo hello world"
                }
            }
        }

        // Build and push Docker image using Kaniko
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

        // Deploy the application to Kubernetes
        stage('Deploy App to Kubernetes') {
          steps {
            container('kubectl') {
              withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://kubernetes.default:443']) {
                sh 'kubectl version --client --short'
                // Update the image tag in myweb.yaml with the build number
                sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
                // Apply the updated manifest to the Kubernetes cluster
                sh 'kubectl apply -f myweb.yaml'
              }
          }
      }
    }
  }
}





// // This is working example of Jenkinsfile for building and deploying Docker image to Kubernetes using Kaniko

// def podTemplate = """
// apiVersion: v1
// kind: Pod
// spec:
//   containers:
//   - name: jenkins-slave
//     image: jenkinsci/slave
//     command:
//     - sleep
//     args:
//     - infinity
//   - name: kubectl
//     image: alpine/k8s:1.24.12
//     command:
//       - /bin/sh
//     tty: true
//   - name: kaniko
//     image: gcr.io/kaniko-project/executor:debug
//     command:
//     - /busybox/cat
//     tty: true
//     volumeMounts:
//       - name: kaniko-secret
//         mountPath: /kaniko/.docker
//   volumes:
//     - name: kaniko-secret
//       secret:
//         secretName: regcred
//         items:
//           - key: .dockerconfigjson
//             path: config.json
// """

// pipeline {
//     agent {
//         kubernetes {
//             yaml podTemplate
//             instanceCap 6
//             defaultContainer 'jenkins-slave'
//         }
//     }

//     stages {
//         stage('Deploy') {
//             steps {
//                 container('jenkins-slave') {
//                     sh "echo hello world"
//                 }
//             }
//         }

//         stage('Kaniko Build & Push Image') {
//             steps {
//                 container('kaniko') {
//                     script {
//                         sh '''
//                             /kaniko/executor --dockerfile `pwd`/Dockerfile \
//                                              --context `pwd` \
//                                              --destination=indamutsa/myweb:${BUILD_NUMBER}
//                         '''
//                     }
//                 }
//             }
//         }

//         stage('Deploy App to Kubernetes') {
//           steps {
//             container('kubectl') {
//               withKubeConfig([credentialsId: 'mykubeconfig', serverUrl: 'https://kubernetes.default:443']) {
//                 sh 'kubectl version --client --short'
//                 sh 'sed -i "s/<TAG>/${BUILD_NUMBER}/" myweb.yaml'
//                 sh 'kubectl apply -f myweb.yaml'
//               }
//           }
//       }
//     }
//   }
// }
