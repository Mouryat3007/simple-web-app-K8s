pipeline {
    agent any

    environment {
        GIT_REPO_URL = 'https://github.com/Mouryat3007/simple-web-app-K8s.git'
        GIT_BRANCH = 'main'
        IMAGE_NAME = 'mouryat3007/webapp'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        FULL_IMAGE = "${IMAGE_NAME}:${IMAGE_TAG}"
        DEPLOYMENT_FILE = 'deployment.yaml'
    }

    stages {
        stage('Clone Git Repo') {
            steps {
                git branch: "${GIT_BRANCH}",
                    credentialsId: 'github-creds',
                    url: "${GIT_REPO_URL}"
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker build -t $FULL_IMAGE .
                        docker push $FULL_IMAGE
                    '''
                }
            }
        }

        stage('Update Image in K8s Manifest') {
            steps {
                sh '''
                    sed -i "s|image: .*|image: ${FULL_IMAGE}|" $DEPLOYMENT_FILE
                '''
            }
        }

        stage('Commit & Push Updated Manifests') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                    sh '''
                        git config user.email "jenkins@automation.com"
                        git config user.name "jenkins"

                        git add $DEPLOYMENT_FILE
                        git commit -m "Update image to ${FULL_IMAGE}"
                        git push https://$GIT_USER:$GIT_PASS@github.com/Mouryat3007/simple-web-app-K8s.git HEAD:$GIT_BRANCH
                    '''
                }
            }
        }

        stage('Sync with ArgoCD (Optional)') {
            steps {
                // Optional: use ArgoCD CLI to force sync
                // Uncomment below if ArgoCD CLI is configured
                // sh 'argocd app sync simple-web-app'
            }
        }
    }
}
