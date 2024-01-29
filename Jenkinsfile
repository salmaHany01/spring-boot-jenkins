pipeline {
    agent any 
    environment {
        DOCKER_REPO = "salmahany3010/jenkins-lab-repo"
        OC_NAMESPACE = "salmahany"
        OC_CONFIG = "/home/salma/Downloads/config"
        OC_SERVER = "https://api.ocpuat.devopsconsulting.org:6443" 
        GITHUB_REPO = "EngMohamedElEmam/new-app"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/${GITHUB_REPO}.git", branch: 'main'
            }
        }
        stage('Build') {
            steps {
                script {
                    def dockerImageName = "${GITHUB_REPO.toLowerCase()}:latest"
                    docker.build(dockerImageName)
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: "jenkins-docker", usernameVariable: "DOCKER_USERNAME", passwordVariable: "DOCKER_PASSWORD")]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push my-image:${BUILD_NUMBER}"
                    }
                }
            }
        }
        stage('Deploy on OC') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'jenkins-oc', variable: 'OPENSHIFT_SECRET')]){
                        sh "oc login -u salmahany -p salmahany https://api.ocpuat.devopsconsulting.org:6443"
                        sh "oc project salmahany"
                        sh "oc new-app \${DOCKER_REPO}:latest"
                    }
                    
                }
            }
        }
    }
    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
