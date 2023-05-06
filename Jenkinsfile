pipeline {
    agent {
        label 'maven'
    }

    parameters {
        string(name:'TAG_NAME',defaultValue: '',description:'')
    }

    environment {
        DOCKER_CREDENTIAL_ID = 'omerevrencomtr'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
        REGISTRY = 'docker.io'
        // need to replace by yourself dockerhub namespace
        DOCKERHUB_NAMESPACE = 'Docker Hub Namespace'
        APP_NAME = 'devops-maven-sample'
        BRANCH_NAME = 'main'
        PROJECT_NAME = 'kubesphere-sample-dev'
    }

    stages {
        stage ('checkout scm') {
            steps {
                // Please avoid committing your test changes to this repository
                git branch: 'master', url: "https://github.com/kubesphere/devops-maven-sample.git"
            }
        }

        stage ('unit test') {
            steps {
                container ('maven') {
                    sh 'mvn clean test'
                }
            }
        }

        stage ('build & push') {
            steps {
                container ('maven') {
                    sh 'mvn -Dmaven.test.skip=true clean package'
                    sh 'docker build -f Dockerfile-online -t $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER .'
                    withCredentials([usernamePassword(passwordVariable : 'DOCKER_PASSWORD' ,usernameVariable : 'DOCKER_USERNAME' ,credentialsId : "$DOCKER_CREDENTIAL_ID" ,)]) {
                        sh 'echo "$DOCKER_PASSWORD" | docker login $REGISTRY -u "$DOCKER_USERNAME" --password-stdin'
                        sh 'docker push  $REGISTRY/$DOCKERHUB_NAMESPACE/$APP_NAME:SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER'
                    }
                }
            }
        }

        stage('deploy to dev') {
             steps {
                 container ('maven') {
                      withCredentials([
                          kubeconfigFile(
                          credentialsId: env.KUBECONFIG_CREDENTIAL_ID,
                          variable: 'KUBECONFIG')
                          ]) {
                          sh 'envsubst < deploy/all-in-one/devops-sample.yaml | kubectl apply -f -'
                      }
                 }
             }
        }
    }
}