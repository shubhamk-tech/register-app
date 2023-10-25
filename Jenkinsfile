pipeline {
    agent any
    tools {
        jdk 'Java17'
        maven 'Maven3'
    }
    
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        APP_NAME = "register-app-pipeline"
        RELEASE = "1.0.0"
        DOCKER_USER = "shubhz1452"
        DOCKER_PASS = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
        JENKINS_API_TOKEN = credentials("JENKINS_API_TOKEN")
    }
    
    stages {
        
        stage ("Cleanup workspaces") {
            steps {
                cleanWs()
            }
        }
        
        stage("Checkoutfrom SCM") {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/shubhamk-tech/register-app.git'
            }
        }
        
        stage ("Build Applicaion") {
            steps {
                sh "mvn clean package"
            }    
        }
        
        stage ("Test Applicaion") {
            steps {
                sh "mvn test"
            }    
        }
        
        stage("Sonarqube Analysis ") {
            steps{
                withSonarQubeEnv('sonarqube-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=register-app \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=register-app '''
                }
            }
        }
        
        stage("Quality Gate") {
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            } 
        }
        
        stage("Build and Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image = docker.build "${IMAGE_NAME}"
                    }
                    
                    docker.withRegistry('',DOCKER_PASS) {
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push('latest')
                    }
                }
            }
        }
        
        stage("Trivy Scan") {
           steps {
               sh "trivy image shubhz1452/register-app-pipeline:latest"
            }
        }
        
        stage("Cleanup Artifacts") {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }
        
        stage("Trigger CD Pipeline") {
            steps {
                script {
                    sh "curl -v -k --user clouduser:${JENKINS_API_TOKEN} -X POST -H 'cache-control: no-cache' -H 'content-type: application/x-www-form-urlencoded' --data 'IMAGE_TAG=${IMAGE_TAG}' 'ec2-52-71-61-56.compute-1.amazonaws.com:8080/job/gitops-register-app-cd/buildWithParameters?token=gitops-token'"
                }
            }
        }
    }
}
