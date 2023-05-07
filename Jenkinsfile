def imageTags = ['latest', '${BUILD_NUMBER}']

pipeline {
    agent {
        node {
            label 'base'
        }
    }

    environment {
        DOCKER_IMAGE = ''
        REGISTRY = 'http://172.16.161.30:5000'
        REGISTRY_CREDENTIAL = 'nexus-admin'
        IMAGE_NAME = 'omerevrencomtr/lamots'
        BUILD_ARGS = "-f Dockerfile . --network host"
        DEPLOY_TAG = "latest"
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    }


    stages {
        stage('Build') {
            steps {
                script {
                    container('base') {
                    withEnv(['DOCKER_TLS_VERIFY=0']) {
                        docker.withRegistry(REGISTRY, REGISTRY_CREDENTIAL) {
                            dockerImage = docker.build(IMAGE_NAME, BUILD_ARGS)
                        }
                        }
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    container('base') {
                        docker.withRegistry(REGISTRY, REGISTRY_CREDENTIAL) {
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