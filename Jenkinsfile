pipeline {
      agent { 
        kubernetes{
            label 'jenkins-slave'
        }
        
    }
    environment {
        IMAGE_NAME = 'rohansharma91/jenkins:${BUILD_NUMBER}'
        
    }

  stages {
    stage('Checkout') {
      steps {
        sh 'echo passed'
        //git branch: 'main', url: 'https://github.com/iam-veeramalla/Jenkins-Zero-To-Hero.git'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "rohansharma91/jenkins:${BUILD_NUMBER}"
        // DOCKERFILE_LOCATION = "java-maven-sonar-argocd-helm-k8s/spring-boot-app/Dockerfile"
        REGISTRY_CREDENTIALS = credentials('docker-cred')
      }
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "docker-cred") {
                dockerImage.push()
            }
        }
      }
    }
    stage ('Image scanning'){
      steps{
        sh 'trivy image $(IMAGE_NAME) > scanning.txt'
      }
    }
    
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "devops-Terraform-pipeline"
            GIT_USER_NAME = "rohansharma91"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "rohan.sharma91@outlook.com"
                    git config user.name "rohansharma91"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s+rohansharma91/jenkins.*+rohansharma91/jenkins:${BUILD_NUMBER}+g" apps/values.yaml
                    git add test/pod.yaml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git pull https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main --allow-unrelated-histories
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
                }
            }
        }
    }
  }
 
