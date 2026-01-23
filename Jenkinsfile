pipeline {
  agent any

  environment {
    IMAGE_NAME = "gowrisuresh0109/github-profile-summarizer-2"
    IMAGE_TAG = "v${env.BUILD_NUMBER}"
    MAX_REPOS = "50"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (Node)') {
      steps {
        sh 'docker run --rm -u $(id -u):$(id -g) -v $PWD:/app -w /app node:20-alpine sh -c "rm -rf node_modules && npm install --cache /tmp/.npm && npm run build"'
       // sh 'npm install'
       // sh 'npm run build'
      }
    }

    stage('Docker Build') {
      steps {
        // SECURELY retrieve the GitHub Token from Jenkins Credentials
        // Make sure you have a Secret Text credential with ID 'github-token-secret'
        withCredentials([string(credentialsId: 'github-token-secret', variable: 'GITHUB_TOKEN_VALUE')]) {
          
          // Pass the token to the Docker build command using --build-arg
          sh "docker build \
            --build-arg VITE_GITHUB_TOKEN=${GITHUB_TOKEN_VALUE} \
            --build-arg VITE_MAX_REPOS=${MAX_REPOS} \
            -t ${IMAGE_NAME}:${IMAGE_TAG} ."
        }
      }
    }

    stage('Docker Login & Push') {
      steps {
        // Assuming 'DockerHub' is the ID for your DockerHub Username/Password credential
        withCredentials([usernamePassword(credentialsId: 'DockerHub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
          sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
        }
        sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
        sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest'
        sh 'docker push $IMAGE_NAME:latest'
      }
    }

  stage('Deploy image'){
    steps{
      sh 'docker run -d -p 8081:80 $IMAGE_NAME:$IMAGE_TAG'
    }
   }
  }

  post {
    always {
      echo "✅ Image pushed: $IMAGE_NAME:$IMAGE_TAG"
    }
  }
}
