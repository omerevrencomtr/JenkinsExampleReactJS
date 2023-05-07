def imageTags = ['latest', '${BUILD_NUMBER}']

pipeline {
    agent {
        node {
            label 'base'
        }
    }

    environment {
        DOCKER_IMAGE = ''
        DOCKER_REGISTRY = 'https://docker-registry.miateknoloji.io'
        REGISTRY_CREDENTIAL = 'nexus-admin'
        IMAGE_NAME = 'omerevrencomtr/lamots'
        BUILD_ARGS = "-f Dockerfile . --network host"
        DEPLOY_TAG = "latest"
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
        SONAR_CREDENTIAL_ID = 'sonar-token'
    }


    stages {
        stage('sonarqube analysis') {
              steps {
                container('maven') {
                  withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('sonar') {
                      sh "mvn sonar:sonar -Dsonar.host.url=http://172.16.161.11:30199 -Dsonar.login=$SONAR_TOKEN"
                    }

                  }
                }

              }
            }

        stage('Build') {
            steps {
                script {
                    container('base') {
                        docker.withRegistry(DOCKER_REGISTRY, REGISTRY_CREDENTIAL) {
                            dockerImage = docker.build(IMAGE_NAME, BUILD_ARGS)
                        }
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    container('base') {
                        docker.withRegistry(DOCKER_REGISTRY, REGISTRY_CREDENTIAL) {
                            for (int i = 0; i < imageTags.size(); ++i) {
                                dockerImage.push("${imageTags[i]}")
                            }
                        }
                    }
                }
            }
        }

        stage ('deploy app') {
            steps {
                script {
                    withCredentials([
                            kubeconfigFile(
                                    credentialsId: KUBECONFIG_CREDENTIAL_ID,
                                    variable: 'KUBECONFIG')
                    ]) {
                        sh 'envsubst < manifest/deploy.yaml | kubectl apply -f -'
                    }
                }
            }
        }

    }
}