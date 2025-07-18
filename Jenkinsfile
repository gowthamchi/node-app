pipeline {
    agent any

    environment {
        IMAGE_NAME = "gowtham1198/node-app"
        GITOPS_REPO = "https://github.com/gowthamchi/node-gitops.git"
        GIT_CREDENTIALS_ID = "gitops-repo"
        DOCKER_CREDENTIALS_ID = "dockerhub-creds"
        TAG = "v${BUILD_NUMBER}"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME:$TAG ."
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${DOCKER_CREDENTIALS_ID}",
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $IMAGE_NAME:$TAG
                    '''
                }
            }
        }

        stage('Update Image in GitOps Repo') {
            steps {
                dir('gitops') {
                    // Clone the GitOps repo using credentials
                    git branch: 'main', credentialsId: "${GIT_CREDENTIALS_ID}", url: "${GITOPS_REPO}"

                    withCredentials([usernamePassword(
                        credentialsId: "${GIT_CREDENTIALS_ID}",
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )]) {
                        script {
                            def deploymentFile = "k8s/deployment.yaml"
                            sh """
                                sed -i 's|image:.*|image: $IMAGE_NAME:$TAG|' $deploymentFile
                                git config user.name 'jenkins'
                                git config user.email 'jenkins@example.com'
                                git add $deploymentFile
                                git commit -m 'Update image to $IMAGE_NAME:$TAG' || echo 'No changes to commit'
                                git push https://$GIT_USER:$GIT_PASS@github.com/gowthamchi/node-gitops.git main
                            """
                        }
                    }
                }
            }
        }
    }
}
