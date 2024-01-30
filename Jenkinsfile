pipeline {
    agent any

    environment {
        
        DOCKER_REGISTRY = "salmahany3010"
        DOCKER_IMAGE = "spring-boot"
        imageTagApp = "build-${BUILD_NUMBER}-app"
        imageNameapp = "${DOCKER_REGISTRY}:${imageTagApp}"
        OPENSHIFT_PROJECT = 'salmahany'
        GITHUB_REPO = "salmaHany01/spring-boot-jenkins"
        OPENSHIFT_SERVER = 'https://api.ocpuat.devopsconsulting.org:6443'
        APP_SERVICE_NAME = 'spring-boot'
        APP_PORT = '8080'
        APP_HOST_NAME = 'springboot.apps.ocpuat.devopsconsulting.org'
        
    }

    stages {
        
        
        stage('Checkout') {
            steps {
                git url: "https://github.com/${GITHUB_REPO}.git", branch: 'main'
            }
        }
        
        stage('Unit Test') {
            steps {
                sh "chmod +x ./gradlew"
                
                sh "./gradlew test --stacktrace"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                    
                sh "chmod +x gradlew"
                    
                sh "docker build -t ${imageNameapp} ."
                
                sh "docker tag ${imageNameapp} docker.io/${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}"
               
            }
        }
        
       
        
        stage('Push Image to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jenkins-docker', usernameVariable: 'DOCKER_REGISTRY_USERNAME', passwordVariable: 'DOCKER_REGISTRY_PASSWORD')]) {
                   
                    sh "echo \${DOCKER_REGISTRY_PASSWORD} | docker login -u \${DOCKER_REGISTRY_USERNAME} --password-stdin"
                    
                    sh "docker push docker.io/${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp}"
                    
                }
            }
        }
        
        

        stage('Deploy to OpenShift') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'openshift-login-token', variable: 'OPENSHIFT_SECRET')]) {
                    sh "oc login --token=\${OPENSHIFT_SECRET} --server=\${OPENSHIFT_SERVER} --insecure-skip-tls-verify"
                    }
                    sh "oc project \${OPENSHIFT_PROJECT}"
                    sh "oc delete dc,svc,deploy,ingress,route \${DOCKER_IMAGE} || true"
                    sh "oc new-app ${DOCKER_REGISTRY}/${DOCKER_IMAGE}:${imageTagApp} --token=\${OPENSHIFT_SECRET}"
                    
                    // Expose the service 
                    sh "oc expose service/${APP_SERVICE_NAME}"
                    
                    //sh "oc create route edge --service \${APP_SERVICE_NAME} --port \${APP_PORT} --hostname springboot.apps.ocpuat.devopsconsulting.org --insecure-policy Redirect"

                }
            }
        }
    }
}
