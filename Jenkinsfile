pipeline {
   agent {
          node {
              label 'base'
          }
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
     imageName = "xpermeet/web"
     buildArgs = "-f Dockerfile . --network host"
     dockerImage = ''
     registryUrl = 'https://docker-push.xperlms.com'
     registryCredential = 'docker-registery'

   }
   stage('Build') {
               steps {
                   script {
                       container('base') {
                           docker.withRegistry(registryUrl, registryCredential) {
                               dockerImage = docker.build(imageName, buildArgs)
                           }
                       }
                   }
               }
   }

   stages {
     stage('docker login') {
       steps{
         container ('nodejs') {
           sh 'echo $DOCKERHUB_CREDENTIAL_PSW  | docker login -u $DOCKERHUB_CREDENTIAL_USR --password-stdin'
         }
       }
     }

     stage('build & push') {
       steps {
         container ('nodejs') {
           sh 'docker build -t $REGISTRY/$DOCKERHUB_USERNAME/$APP_NAME .'
           sh 'docker push $REGISTRY/$DOCKERHUB_USERNAME/$APP_NAME'
         }
       }
     }
     stage ('deploy app') {
       steps {
          container ('nodejs') {
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