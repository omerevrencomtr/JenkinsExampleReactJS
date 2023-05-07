pipeline {
  agent {
    node {
      label 'maven'
    }

  }
  parameters {
        string(name:'BRANCH_NAME',defaultValue: 'master',description:'')
    }
  environment {
        DOCKER_CREDENTIAL_ID = 'dockerhub-omerevrencomtr'
        PROD_KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
        TEST_KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
        DEV_KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'

        REGISTRY = 'docker.io'
        DOCKERHUB_NAMESPACE = 'omerevrencomtr'
        APP_NAME = 'devops-maven-sample'
        SONAR_CREDENTIAL_ID = 'sonar-token'
        TAG_NAME = "SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
    }
  stages {
    stage('checkout') {
      steps {
        container('maven') {
          git branch: 'master', url: 'https://github.com/kubesphere/devops-maven-sample.git'
        }
      }
    }
    stage('unit test') {
      steps {
        container('maven') {
          sh 'mvn clean test'
        }
      }
    }
    stage('sonarqube analysis') {
      steps {
        container('maven') {
          withCredentials([string(credentialsId: "$SONAR_CREDENTIAL_ID", variable: 'SONAR_TOKEN')]) {
            withSonarQubeEnv('sonar') {
              sh "mvn sonar:sonar -Dsonar.login=$SONAR_TOKEN"
            }

          }
        }

      }
    }
    stage('build & push') {
      steps {
        container('maven') {
          sh 'mvn -Dmaven.test.skip=true clean package'
          sh 'docker build -f Dockerfile-online -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
          withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
            sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
            sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
          }
        }
      }
    }
    stage('push latest') {
      steps {
        container('maven') {
          sh 'docker tag  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
          sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:latest '
        }
      }
    }
    stage('deploy to dev') {
      steps {
         container('maven') {
            withCredentials([
                kubeconfigFile(
                credentialsId: env.DEV_KUBECONFIG_CREDENTIAL_ID,
                variable: 'KUBECONFIG')
                ]) {
                sh 'envsubst < deploy/dev-all-in-one/devops-sample.yaml | kubectl apply -f -'
            }
         }
      }
    }
    stage('deploy to staging') {
      steps {
         container('maven') {
            input(id: 'deploy-to-staging', message: 'deploy to staging?')
            withCredentials([
                kubeconfigFile(
                credentialsId: env.TEST_KUBECONFIG_CREDENTIAL_ID,
                variable: 'KUBECONFIG')
                ]) {
                sh 'envsubst < deploy/prod-all-in-one/devops-sample.yaml | kubectl apply -f -'
            }
         }
      }
    }
    stage('deploy to production') {
      steps {
         container('maven') {
            input(id: 'deploy-to-production', message: 'deploy to production?')
            withCredentials([
                kubeconfigFile(
                credentialsId: env.PROD_KUBECONFIG_CREDENTIAL_ID,
                variable: 'KUBECONFIG')
                ]) {
                sh 'envsubst < deploy/prod-all-in-one/devops-sample.yaml | kubectl apply -f -'
            }
         }
      }
    }
  }
}