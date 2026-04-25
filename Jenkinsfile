pipeline {
    agent any

    environment {
        // We will configure these credentials securely in Jenkins later
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
        GITHUB_CREDENTIALS = credentials('github-creds')
        
        // Use the Jenkins build number as the unique image tag
        IMAGE_TAG = "v${env.BUILD_NUMBER}" 
        DOCKERHUB_USER = "mehboob16" // REPLACE THIS
        MANIFEST_REPO = "github.com/mehboob16/voting-app-manifests.git" // REPLACE THIS
    }

    stages {
        stage('Build Docker Images') {
            steps {
                script {
                    echo "Building Images with tag ${IMAGE_TAG}..."
                    sh "docker build -t ${DOCKERHUB_USER}/vote:${IMAGE_TAG} ./vote"
                    sh "docker build -t ${DOCKERHUB_USER}/result:${IMAGE_TAG} ./result"
                    sh "docker build -t ${DOCKERHUB_USER}/worker:${IMAGE_TAG} ./worker"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo "Logging into DockerHub..."
                    sh "echo \$DOCKERHUB_CREDENTIALS_PSW | docker login -u \$DOCKERHUB_CREDENTIALS_USR --password-stdin"
                    
                    echo "Pushing Images..."
                    sh "docker push ${DOCKERHUB_USER}/vote:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/result:${IMAGE_TAG}"
                    sh "docker push ${DOCKERHUB_USER}/worker:${IMAGE_TAG}"
                }
            }
        }

        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    echo "Cloning Manifest Repository..."
                    sh "git clone https://${GITHUB_CREDENTIALS_USR}:${GITHUB_CREDENTIALS_PSW}@${MANIFEST_REPO} manifests-repo"
                    
                    dir('manifests-repo') {
                        echo "Updating Image Tags in YAML files..."
                        // Using sed to replace the old tag with the new Jenkins build tag
                        sh "sed -i 's|image: ${DOCKERHUB_USER}/vote:.*|image: ${DOCKERHUB_USER}/vote:${IMAGE_TAG}|g' vote.yaml"
                        sh "sed -i 's|image: ${DOCKERHUB_USER}/result:.*|image: ${DOCKERHUB_USER}/result:${IMAGE_TAG}|g' result.yaml"
                        sh "sed -i 's|image: ${DOCKERHUB_USER}/worker:.*|image: ${DOCKERHUB_USER}/worker:${IMAGE_TAG}|g' worker.yaml"
                        
                        echo "Committing and Pushing to trigger ArgoCD..."
                        sh "git config user.email 'jenkins@automation.local'"
                        sh "git config user.name 'Jenkins CI'"
                        sh "git add ."
                        sh "git commit -m 'ci: Update image tags to ${IMAGE_TAG}'"
                        sh "git push origin main"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // Clean up Docker images from the Jenkins server to save space
            sh "docker rmi ${DOCKERHUB_USER}/vote:${IMAGE_TAG} || true"
            sh "docker rmi ${DOCKERHUB_USER}/result:${IMAGE_TAG} || true"
            sh "docker rmi ${DOCKERHUB_USER}/worker:${IMAGE_TAG} || true"
            cleanWs()
        }
    }
}