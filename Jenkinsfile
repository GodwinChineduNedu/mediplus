pipeline {
  agent {
    docker {
      image 'maven:3.8.6-openjdk-11'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
  }

  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/GodwinChineduNedu/mediplus'
      }
    }

    stage('Build and Test') {
      steps {
        sh 'ls -ltr'
        sh 'cd java-maven-sonar-argocd-helm-k8s/mediplus-app && mvn clean package'
      }
    }

    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://13.222.28.200:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'cd java-maven-sonar-argocd-helm-k8s/mediplus-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }

    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "godwinchinedu/mediplus-app:${BUILD_NUMBER}"
      }
      steps {
        script {
          sh 'cd java-maven-sonar-argocd-helm-k8s/mediplus-app && docker build -t ${DOCKER_IMAGE} .'
          docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
            sh 'docker push ${DOCKER_IMAGE}'
          }
        }
      }
    }

    stage('Update Deployment File') {
      steps {
        withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
          script {
            sh '''
              git config --global user.email "godwinchinedunedu@gmail.com"
              git config --global user.name "GodwinChineduNedu"
              sed -i -E "s/replaceImageTag/${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
              git add .
              git commit -m "Update image tag to ${BUILD_NUMBER}" || echo "No changes"
              git push origin main || echo "Push failed"
            '''
          }
        }
      }
    }
  }
}
