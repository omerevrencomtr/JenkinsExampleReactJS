pipeline {
   agent {
     label 'go'
   }

   environment {
     // the address of your Docker Hub registry
     REGISTRY = 'docker.io'
     // your Docker Hub username
     DOCKERHUB_USERNAME = 'Docker Hub Username'
     // Docker image name
     APP_NAME = 'devops-go-sample'
     // 'dockerhubid' is the credentials ID you created in KubeSphere with Docker Hub Access Token
     DOCKERHUB_CREDENTIAL = credentials('dockerhub-omerevrencomtr')
     // the kubeconfig credentials ID you created in KubeSphere
     KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
     // the name of the project you created in KubeSphere, not the DevOps project name
     PROJECT_NAME = 'devops-go'
   }

   stages {
     stage('docker login') {
       steps{
         container ('go') {
           sh 'echo $DOCKERHUB_CREDENTIAL_PSW  | docker login -u $DOCKERHUB_CREDENTIAL_USR --password-stdin'
         }
       }
     }

     stage('build & push') {
       steps {
         container ('go') {
           sh 'git clone https://github.com/yuswift/devops-go-sample.git'
           sh 'cd devops-go-sample && docker build -t $REGISTRY/$DOCKERHUB_USERNAME/$APP_NAME .'
           sh 'docker push $REGISTRY/$DOCKERHUB_USERNAME/$APP_NAME'
         }
       }
     }
     stage ('deploy app') {
       steps {
          container ('go') {
             withCredentials([
               kubeconfigFile(
                 credentialsId: env.KUBECONFIG_CREDENTIAL_ID,
                 variable: 'KUBECONFIG')
               ]) {
               sh 'envsubst < devops-go-sample/manifest/deploy.yaml | kubectl apply -f -'
             }
          }
       }
    }
  }
}