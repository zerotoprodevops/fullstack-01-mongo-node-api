pipeline{
    agent{
        label  "build-vm"
    }
    triggers{
        pollSCM('H/5 * * * *')
    }


    environment {
        IMAGE = 'jyotimanab/fullstack-01-mongo-node-api'
        TAG =  "${env.BUILD_NUMBER}"
    }
    stages{
        stage('git checkout'){
            steps{
                git branch: 'main',
                    url: 'https://github.com/zerotoprodevops/fullstack-01-mongo-node-api.git'
            }
        }   
        stage('build'){
            steps{
                sh "docker build -t ${IMAGE}:${TAG} . "
            }
        }    
        stage('test'){
            steps{
                sh "TMPDIR=/tempdir trivy image ${IMAGE}:${TAG}"
            }           
        }
        stage('Push toDocker hub'){
            steps{
                withDockerRegistry(credentialsId: 'dockerhub-cred', url: ''){
                sh """
                docker push ${IMAGE}:${TAG}
                docker tag ${IMAGE}:${TAG} ${IMAGE}:latest
                docker push ${IMAGE}:latest
                """}
            }
        }
        stage('Update GitOps Manifests'){
            steps{
                withCredentials([usernamePassword(credentialsId: 'github-token', passwordVariable: 'GIT_PASS', usernameVariable: 'GIT_USER')]){
                    sh """
                        # 1. Clone the Config Repo
                        rm -rf fullstack-01-gitops-configs
                        git clone https://${GIT_USER}:${GIT_PASS}@github.com/zerotoprodevops/fullstack-01-gitops-configs.git
                        cd fullstack-01-gitops-configs
                        
                        # 2. Update the image tag in the API values.yaml
                        # (Notice the path: api-chart/values.yaml)
                        sed -i "s|  tag: .*|  tag: \\"${TAG}\\"|g" api-chart/values.yaml
                        
                        # 3. Commit and Push
                        git config user.email "jenkins@k8cluster.local"
                        git config user.name "Jenkins CI"
                        git add .
                        git commit -m "Update API image tag to ${TAG} [skip ci]"
                        git push origin main
                    """
                }
            }

        }

    }
    post{
        success {
            echo "Build ${TAG} successful - Image on Gicker hub"
            sh "docker rmi ${IMAGE}:${TAG} ${IMAGE}:latest || echo 'cleanup: Image not found locally'"
        }
        failure{
            echo "pipeline failed - Image kept locally for debugging"
        }
    }
        
}