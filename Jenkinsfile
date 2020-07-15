pipeline {
    agent any
    environment {
      // Configure in global tool setting of jenkins slave or master
        CMDLINE = "clean initialize package"
        DEV_ECR_URL = "ip-172-31-45-42.ap-south-1.compute.internal:5000/sample"
        POD_PATTERN = "sample-"
        K8S_NAMESPACE = "tibco"
        DOCKER_IMAGE = "sample"
    }
    stages {
      stage('Cleanup')
      {
        steps {
          echo "Cleaning it up..."
          echo currentBuild.projectName
        }
      }
      stage('Docker Build') 
      {
        steps {
          echo "Compile & Build..."
          sh 'mvn -f restapp1.parent/pom.xml clean initialize package docker:build'
        }
      }
      stage('Docker Push') 
      {
        steps {
          echo "Deploying..."
          sh '''
            export THISTAG=`date "+%y%m%d%H%M"`

            docker tag $DOCKER_IMAGE:latest $DEV_ECR_URL:$THISTAG
            docker push $DEV_ECR_URL:$THISTAG
          '''
        }
      }
      stage('Container Restart') 
      {
        steps {
          echo "Restarting container"
          sh '''
            export PODNAME=`kubectl -n $K8S_NAMESPACE get pods | grep $POD_PATTERN | awk '{print $1}'`
            kubectl -n $K8S_NAMESPACE delete pod $PODNAME
          '''
          }
      }
      stage('QA Test') 
      {
        steps {
          echo "QA Test..."
        }
      }
    }
    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '25', artifactNumToKeepStr: '25'))
    }
}
