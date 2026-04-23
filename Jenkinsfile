pipeline {
  agent any   // ✅ run on Jenkins node (must have docker + mvn installed)

  stages {

    stage('Checkout') {
      steps {
        git(
          branch: 'main',
          url: 'https://github.com/yesk993-ops/Java-based-App.git',
          credentialsId: 'github'
        )
      }
    }

    stage('Build and Test') {
      steps {
        dir('spring-boot-app') {
          sh 'mvn clean package'
        }
      }
    }

    stage('Static Code Analysis') {
      environment {
        SONAR_URL = "http://192.168.88.128:9000/"
      }
      steps {
        dir('spring-boot-app') {
          withCredentials([string(credentialsId: 'sonarqube', variable: 'SONAR_AUTH_TOKEN')]) {
            sh '''
              mvn sonar:sonar \
              -Dsonar.login=$SONAR_AUTH_TOKEN \
              -Dsonar.host.url=$SONAR_URL
            '''
          }
        }
      }
    }

    stage('Build and Push Docker Image') {
      environment {
        DOCKER_IMAGE = "mydocker3692/spring-boot-app:${BUILD_NUMBER}"
      }
      steps {
        script {
          dir('spring-boot-app') {
            sh "docker build -t ${DOCKER_IMAGE} ."
          }

          docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-cred') {
            sh "docker push ${DOCKER_IMAGE}"
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
        withCredentials([string(credentialsId: 'git', variable: 'GITHUB_TOKEN')]) {
          sh '''
            git config user.email "jenkins@example.com"
            git config user.name "Jenkins"

            sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" spring-boot-app-manifests/deployment.yml
            sed -i "s|abhishekf5/ultimate-cicd|mydocker3692/spring-boot-app|g" spring-boot-app-manifests/deployment.yml

            git add spring-boot-app-manifests/deployment.yml
            git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes"

            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        echo "Deploying: mydocker3692/spring-boot-app:${BUILD_NUMBER}"
        // sh 'kubectl apply -f spring-boot-app-manifests/'
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
