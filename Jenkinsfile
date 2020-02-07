pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t anilkumblepuli/vprofile:$BUILD_NUMBER "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'dokerHub_Pass', variable: 'dokerHub_Pass')]) {
                    sh "docker login -u anilkumblepuli -p ${dokerHub_Pass}"
                    sh "docker push anilkumblepuli/vprofile:$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh $BUILD_NUMBER"
                sshagent(['kops-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ubuntu@52.66.195.219:/home/ubuntu/"
                    script{
                        try{
                            sh "ssh ubuntu@52.66.195.219 kubectl apply -f ."
                        }catch(error){
                            sh "ubuntu@52.66.195.219 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
