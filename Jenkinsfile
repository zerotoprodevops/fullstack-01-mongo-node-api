pipeline{
    agent{
        label  "build-vm"
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
                withDockerRegistry(credentialsId: 'dockerhub-cred', url: '')
                sh """
                docker push ${IMAGE}:${TAG}
                docker tag ${IMAGE}:${TAG} ${IMAGE}:latest
                docker push ${IMAGE}:latest
                """
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