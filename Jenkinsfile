pipeline {
    agent any
    tools { 
        maven 'M3' 
    }
    options {
        buildDiscarder logRotator( 
            daysToKeepStr: '2', 
            numToKeepStr: '2'
        )
    }
    environment {
        DOCKERHUB_USERNAME = "harsha2893"
        JOB_NAME = "spring-petclinic"
        APP_NAME = "spring-petclinic"
        IMAGE_TAG = "${BUILD_NUMBER}"
        IMAGE_NAME = "${DOCKERHUB_USERNAME}" + "/" + "${APP_NAME}"
        REGISTRY_CREDS = 'dockerhub'
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                script {
                    cleanWs()
                    sh """
                    echo "Cleaned Up Workspace for ${JOB_NAME}"
                    """
                }
            }
        }
        stage('Checkout SCM'){
            steps {
                git url: 'https://github.com/harsha19932893/demo.git',
                branch: 'main'
            }
        }
        stage('Code Build') {
            steps {
                 sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
            post {
                success {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
        stage('Build Docker Image'){
            steps {
                script{
                    docker_image = docker.build "${IMAGE_NAME}"
                }
            }
        }
        stage('Push Docker Image'){
            steps {
                script{
                    docker.withRegistry('', REGISTRY_CREDS ){
                        docker_image.push("${BUILD_NUMBER}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        stage('Start MiniKube'){
            steps {
                sh "minikube start"
            }
        }  
        stage('Deploy new image to Minikube') {
            steps {
                sh "kubectl create deployment petclinic-minikube --image=${IMAGE_NAME}:${IMAGE_TAG}"
                sh "kubectl expose deployment petclinic-minikube --type=NodePort --port=8181"
                sh "kubectl get services petclinic-minikube"
            }
        }
    }   
}
