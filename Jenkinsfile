def imageTags = ['latest', '${BUILD_NUMBER}']

pipeline {
    agent {
        node {
            label 'base'
        }
    }

    environment {
        DOCKER_IMAGE = ''
        REGISTRY = ''
        REGISTRY_CREDENTIAL = 'dockerhub-omerevrencomtr'
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
                        docker.withRegistry(registryUrl, registryCredential) {
                            dockerImage = docker.build(imageName, buildArgs)
                        }
                    }
                }
            }
        }

        stage('Push') {
            steps {
                script {
                    container('base') {
                        docker.withRegistry(registryUrl, registryCredential) {
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
                                    credentialsId: env.KUBECONFIG_CREDENTIAL_ID,
                                    variable: 'KUBECONFIG')
                    ]) {
                        sh 'envsubst < manifest/deploy.yaml | kubectl apply -f -'
                    }
                }
            }
        }

    }
}