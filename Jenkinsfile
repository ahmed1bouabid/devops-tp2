pipeline {
  agent any
  options { timestamps() }

  environment {
    IMAGE = 'ahmedbouabid/monapp'
    TAG   = "build-${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }   // lit le même repo que le job
    }

    stage('Docker Build') {
      steps {
        sh 'docker version'
        sh 'docker build -t $IMAGE:$TAG .'
      }
    }

    stage('Smoke Test') {
      steps {
        sh '''
          docker rm -f monapp_test 2>/dev/null || true
          docker run -d --name monapp_test -p 8081:80 $IMAGE:$TAG
          sleep 3
          curl -sI http://localhost:8081 | grep "200 OK"
          docker rm -f monapp_test
        '''
      }
    }

    stage('Push (Docker Hub)') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                          usernameVariable: 'USER',
                                          passwordVariable: 'PASS')]) {
          sh '''
            echo "$PASS" | docker login -u "$USER" --password-stdin
            docker tag $IMAGE:$TAG $IMAGE:latest
            docker push $IMAGE:$TAG
            docker push $IMAGE:latest
          '''
        }
      }
    }
  }

  post {
    success { echo 'Build+Test+Push OK' }
    failure { echo 'Build/Tests/Push KO' }
  }
}
