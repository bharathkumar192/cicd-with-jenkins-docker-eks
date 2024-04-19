pipeline {
    agent any

    stages {
        stage('Clone git repo') {
            steps {
                sh 'echo "STAGE 0: Cloning app code from SCM ..."'
                git 'https://github.com/bharathkumar192/cicd-with-jenkins-docker-eks.git'
            }    
        }        
        stage('Lint all app code') {
            steps {
                sh 'echo "STAGE 1: Checking app code for syntax error ..."'
                sh 'tidy -q -e *.html || true'
            }
        }   
        stage( 'Build docker image for app' ) {
            steps {
                sh 'echo "STAGE 2: Building and tagging docker image ..."'
                sh 'docker build -t web-app:v1.0 .'
                sh 'docker image ls'                  
            }
        } 
        stage('Push image to DockerHub repo') {
            steps {
                withDockerRegistry([url: "https://index.docker.io/v1/", credentialsId: "dockerhub-token"]) {
                    sh 'echo "STAGE 3: Uploading image to DockerHub repository ..."'
                    script {
                        try {
                            sh 'docker tag web-app:v1.0 bharathkumar192/web-app:v1.0'
                            sh 'docker push bharathkumar192/web-app:v1.0'
                        } catch (Exception e) {
                            sh 'echo "Failed to push Docker image. Error: $e"'
                            error "Stopping the build due to failure in pushing Docker image."
                        }
                    }
                }
            }
        }                   
        stage('Deploy image to AWS EKS') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'eu-north-1') {
                    sh 'echo "STAGE 4: Deploying image to AWS EKS cluster ..."'
                    sh 'aws eks update-kubeconfig --name JenkinsApp'
                    sh 'kubectl config use-context arn:aws:eks:eu-north-1:730335486616:cluster/JenkinsApp'
                    sh 'kubectl apply -f templates/aws-auth-cm.yml'
                    sh 'kubectl apply -f templates/deployment.yml'
                    sh 'kubectl apply -f templates/loadbalancer.yml'
                    sh 'kubectl set image deployment/web-app web-app=bharathkumar192/web-app:v1.0'
                    sh 'kubectl rollout status deployment web-app'
                    sh 'kubectl get nodes --all-namespaces'
                    sh 'kubectl get deployment'
                    sh 'kubectl get pod -o wide'
                    sh 'kubectl get service/web-app'
                    sh 'echo "Congratulations! Deployment successful."'
                    sh 'kubectl describe deployment web-app'
                }
            }
        }

           
    }
}
