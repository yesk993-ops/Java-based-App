pipeline {
  agent {
    docker {
      image 'maven:3.8.4-openjdk-11'
      args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
    }
  }
  stages {
    stage('Checkout') {
      steps {
        git branch: 'main', url: 'https://github.com/yesk993-ops/Java-based-App.git'
        credentialsId: 'github'
      }
    }
    stage('Build and Test') {
      steps {
        // build the project and create a JAR file
        sh 'mvn clean package'
      }
    }
    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://34.201.116.83:9000"
      }
      steps {
        withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
          sh 'mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=${SONAR_URL}'
        }
      }
    }
    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "mydocker3692/spring-boot-app:${BUILD_NUMBER}"
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
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Java-based-App"
            GIT_USER_NAME = "yesk993-ops"
        }
        steps {
            withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "jenkins@example.com"
                    git config user.name "Jenkins"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
                    sed -i "s|abhishekf5/ultimate-cicd|mydocker3692/spring-boot-app|g" spring-boot-app-manifests/deployment.yml
                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    }
    stage('Deploy to Kubernetes') {
      steps {
        script {
          // This stage would deploy to Kubernetes using kubectl or Helm
          // For now, we'll just echo the deployment command
          echo "Deploying to Kubernetes with image: mydocker3692/spring-boot-app:${BUILD_NUMBER}"
          // Example command:
          // sh 'kubectl apply -f spring-boot-app-manifests/'
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
