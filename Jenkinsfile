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
        DOCKER_IMAGE = "mydocker3692/spring-boot-app:latest"
      }
      steps {
        script {
          dir('spring-boot-app') {
            // build with both tags (latest + build number)
            sh "docker build -t ${DOCKER_IMAGE} -t mydocker3692/spring-boot-app:${BUILD_NUMBER} ."
          }

          docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-cred') {
            sh "docker push ${DOCKER_IMAGE}"
            sh "docker push mydocker3692/spring-boot-app:${BUILD_NUMBER}"
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

            # Always set manifest to use latest tag
            sed -i "s|image:.*|image: mydocker3692/spring-boot-app:latest|g" spring-boot-app-manifests/deployment.yml

            git add spring-boot-app-manifests/deployment.yml
            git commit -m "Update deployment image to latest" || echo "No changes"

            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
          '''
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        withEnv(["KUBECONFIG=/var/lib/jenkins/.kube/config"]) {
          sh 'kubectl apply -f spring-boot-app-manifests/'
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
