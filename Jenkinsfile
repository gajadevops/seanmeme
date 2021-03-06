pipeline {
  environment {
    IMAGE_NAME = 'demohelm'
  }

  agent any

  stages {
    stage('Build') {
      steps {
        checkout scm
        sh '''
          docker run --rm -v "${PWD}":/go/src/seanmeme -w /go/src/seanmeme -e CGO_ENABLED=0 golang:1.8 go build
          docker build -t $IMAGE_NAME:$BUILD_ID .
        '''
      }
    }
    stage('Test') {
      steps {
        echo 'TODO: add tests'
      }
    }
    stage('Image Release') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }

      steps {
          sh '''
            echo $(aws ecr get-login --region us-east-2 --registry-ids 410602862282) > file.txt
            $( sed "s/-e none//g" file.txt)
            sudo docker tag $IMAGE_NAME:${BUILD_ID} 410602862282.dkr.ecr.us-east-2.amazonaws.com/demo-poc:${BUILD_ID}
            sudo docker push 410602862282.dkr.ecr.us-east-2.amazonaws.com/demo-poc:${BUILD_ID}
          '''
        }
      }
    stage('Staging Deployment') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }

      environment {
        RELEASE_NAME = 'demo-staging'
        SERVER_HOST = 'staging.sogendh.com'
      }

      steps {
        sh '''
          helm upgrade --install --namespace staging $RELEASE_NAME ./helm/seanmeme --set image.tag=$BUILD_ID,ingress.host=$SERVER_HOST
        '''
      }
    }
    stage('Deploy to Production?') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }

      steps {
        // Prevent any older builds from deploying to production
        milestone(1)
        input 'Deploy to Production?'
        milestone(2)
      }
    }
    stage('Production Deployment') {
      when {
        expression { env.BRANCH_NAME == 'master' }
      }

      environment {
        RELEASE_NAME = 'demo-production'
        SERVER_HOST = 'production.sogendh.com'
      }

      steps {
        sh '''
          helm upgrade --install --namespace production $RELEASE_NAME ./helm/seanmeme --set image.tag=$BUILD_ID,ingress.host=$SERVER_HOST
        '''
      }
    }
  }

}

