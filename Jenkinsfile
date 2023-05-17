// Define environment variables
env.dockerimagename = "maxbova/http_server"
env.docker_build_number = "commit_verification"
def dockerImage

node {
    // Checkout
    stage('Clone the json server project') {
        checkout([$class: 'GitSCM', branches: [[name: "${env.ghprbActualCommit}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/Maxayer/http_file_server.git']]])

    }
    
    // Build Docker image
    stage('Build image') {
        dockerImage = docker.build env.dockerimagename
    }

    // Push Docker image
    stage('Pushing Image') {
        withCredentials([usernamePassword(credentialsId: 'docker_entry', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
            docker.withRegistry('https://registry.hub.docker.com', 'docker_entry') {
                dockerImage.push("commit_verification")
            }
        }
    }

    // Remove Docker image
    stage('Remove Unused docker image') {
        sh "docker rmi -f ${env.dockerimagename}:${env.docker_build_number}"
    }

    //Deploy to minikube
    stage('Deploying App to Kubernetes') {
        kubernetesDeploy(configs: "json_server.yaml", kubeconfigId: "kuber_entry")
    }
    
    //check containerized application
    stage('Run Tests') {
        checkout([
            $class: 'GitSCM', 
            branches: [[name: '*/main']],
            doGenerateSubmoduleConfigurations: false, 
            extensions: [], 
            submoduleCfg: [], 
            userRemoteConfigs: [[url: 'https://github.com/Maxayer/JsonBenchmarking.git']]
        ])
        def minikubeIP = sh(script: "minikube ip", returnStdout: true).trim()
        sh "gradle gatlingRun-JsonServerTest -DminikubeIP=${minikubeIP}"
    }
}

