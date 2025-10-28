pipeline {
  agent { label 'agent' }
  environment {
    PROJECT = 'crested-polygon-472204-n5'
    REGION = 'us-central1'
    CLUSTER = 'cluster-1'
    IMAGE = 'us-central1-docker.pkg.dev/crested-polygon-472204-n5/jenkins-docker-repo/python-app'
  }
  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Auth to GCP') {
      steps {
        withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
            gcloud config set project $PROJECT
            gcloud auth configure-docker $REGION-docker.pkg.dev --quiet
          '''
        }
      }
    }

    stage('Build & Push Image') {
      steps {
        sh '''
          TAG=${BUILD_NUMBER}
          docker build -t $IMAGE:$TAG .
          docker push $IMAGE:$TAG
          echo "$IMAGE:$TAG" > image.txt
        '''
      }
    }

    stage('Deploy to GKE') {
      steps {
        withCredentials([file(credentialsId: 'gcp-sa-key', variable: 'GOOGLE_APPLICATION_CREDENTIALS')]) {
          sh '''
            gcloud auth activate-service-account --key-file=$GOOGLE_APPLICATION_CREDENTIALS
            gcloud container clusters get-credentials $CLUSTER --region $REGION
            IMAGE_TAG=$(cat image.txt)
            sed "s|IMAGE_PLACEHOLDER|$IMAGE_TAG|" k8s/deployment.yaml | kubectl apply -f -
            kubectl apply -f k8s/service.yaml
            kubectl rollout status deployment/python-app
          '''
        }
      }
    }
  }
}
