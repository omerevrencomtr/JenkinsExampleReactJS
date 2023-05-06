pipeline {
   agent {
     label 'node'
   }

   environment {
     // the address of your Docker Hub registry
     REGISTRY = 'https://index.docker.io/v2/'
     // your Docker Hub username
     DOCKERHUB_USERNAME = 'omerevrencomtr'
     // Docker image name
     APP_NAME = 'JenkinsExampleReactJs'
     // 'dockerhubid' is the credentials ID you created in KubeSphere with Docker Hub Access Token
     DOCKERHUB_CREDENTIAL = credentials('dockerhub-omerevrencomtr')
     // the kubeconfig credentials ID you created in KubeSphere
     KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
     // the name of the project you created in KubeSphere, not the DevOps project name
     PROJECT_NAME = 'lamots-dev'
   }

   stages {
     stage('docker login') {
       steps{
         container ('node') {
           sh 'echo $DOCKERHUB_CREDENTIAL_PSW  | crictl login -u $DOCKERHUB_CREDENTIAL_USR --password-stdin'
         }
       }
     }

     stage('build & push') {
       steps {
         container ('node') {
           sh 'containerd build -t $REGISTRY/$DOCKERHUB_USERNAME/$APP_NAME .'
           sh 'containerd push $REGISTRY/$DOCKERHUB_USERNAME/$APP_NAME'
         }
       }
     }
     stage ('deploy app') {
       steps {
          container ('node') {
             withCredentials([
               kubeconfigFile(
                 credentialsId: env.KUBECONFIG_CREDENTIAL_ID,
                 variable: 'KUBECONFIG')
               ]) {
               sh 'envsubst < JenkinsExampleReactJS/manifest/deploy.yaml | kubectl apply -f -'
             }
          }
       }
    }
  }
}