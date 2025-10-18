pipeline {
    agent {
        kubernetes {
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              name: gitops-agent
            spec:
              containers:
              - name: git
                image: alpine/git:2.45
                command:
                - cat
                tty: true
            '''
        }
    }

    parameters {
        string(name: 'IMAGE_TAG', defaultValue: '', description: 'Docker 이미지 태그 (빌드 번호)')
    }

    environment {
        GITHUB_CREDENTIALS_ID = 'github-credentials'
        REPO_URL = 'https://github.com/iiijong/stock101-k8s-manifests.git'
        DEPLOYMENT_FILE = 'k8s/frontend/deployment.yaml'
    }

    stages {
        stage('Clone GitOps Repository') {
            steps {
                container('git') {
                    withCredentials([usernamePassword(
                        credentialsId: GITHUB_CREDENTIALS_ID,
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_TOKEN'
                    )]) {
                        sh '''
                            echo "Cloning GitOps repository..."
                            rm -rf repo
                            git clone https://$GIT_USER:$GIT_TOKEN@github.com/iiijong/stock101-k8s-manifests.git repo
                            cd repo
                            git checkout main
                            git pull origin main --rebase
                        '''
                    }
                }
            }
        }

        stage('Update Deployment YAML') {
            steps {
                dir('repo') {
                    container('git') {
                        sh '''
                            echo "Updating image tag..."
                            sed -i "s|iiijong/frontend:.*|iiijong/frontend:${IMAGE_TAG}|g" ${DEPLOYMENT_FILE}
                            grep "image:" ${DEPLOYMENT_FILE}
                        '''
                    }
                }
            }
        }

        stage('Commit & Push Changes') {
            steps {
                dir('repo') {
                    container('git') {
                        sh '''
                            echo "Committing and pushing changes..."
                            git config user.name "iiijong"
                            git config user.email "pjwfish@naver.com"
                            git add ${DEPLOYMENT_FILE}
                            git commit -m "update frontend image to ${IMAGE_TAG}" || echo "No changes to commit"
                            git pull origin main --rebase
                            git push origin main
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "GitOps update completed successfully."
        }
        failure {
            echo "GitOps update failed."
        }
    }
}
