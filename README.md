pipeline {
agent any
environment {
registry = "971627905827.dkr.ecr.us-east-1.amazonaws.com/demo-repo"
}
stages {
stage('Cloning Git') {
steps {
checkout([$class: 'GitSCM', branches: [[name: '*/master']],
doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [],
userRemoteConfigs: [[credentialsId: '', url:
'https://github.com/akannan1087/myPythonDockerRepo']]])
}
}
// Building Docker images
stage('Building image') {
steps{
script {
sh 'docker build -t demo-repo -f /var/lib/jenkins/workspace/test_ecr_repo/Dockerfile .'
}
}
}
stage('Tagging image') {
steps{
script {
sh 'docker tag demo-repo:latest
971627905827.dkr.ecr.us-east-1.amazonaws.com/demo-repo:latest'
}
}
}
// Uploading Docker images into AWS ECR
stage('Pushing to ECR') {
steps{
script {
sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS
--password-stdin 971627905827.dkr.ecr.us-east-1.amazonaws.com'
sh 'docker push 971627905827.dkr.ecr.us-east-1.amazonaws.com/demo-repo:latest'
}
}
}
// Stopping Docker containers for cleaner Docker run
stage('stop previous containers') {
steps {
sh 'docker ps -f name=mypythonContainer -q | xargs --no-run-if-empty docker container
stop'
sh 'docker container ls -a -fname=mypythonContainer -q | xargs -r docker container rm'
}
}
stage('Docker Run') {
steps{
script {
sh 'docker run -d -p 8096:5000 --rm --name mypythonContainer
971627905827.dkr.ecr.us-east-1.amazonaws.com/demo-repo:latest'
}
}
}
}
}
