pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  tools {
    node 'node-latest'
  }
  parameters {
    string(name: 'IMAGE_REPO_NAME', defaultValue: 'jamessmith52963/basic-react', description: '')
    string(name: 'LATEST_BUILD_TAG', defaultValue: 'build-latest', description: '')
    string(name: 'DOCKER_COMPOSE_FILENAME', defaultValue: 'docker-compose.yml', description: '')
    string(name: 'DOCKER_STACK_NAME', defaultValue: 'react_stack', description: '')
    booleanParam(name: 'DOCKER_STACK_RM', defaultValue: 'false', description: 'Remove previous stack.  This is required if you have updated any secrets or configs as these cannot be updated. ')
  }
  stages {
    stage('npm install'){
      steps{
        sh "npm install"
      }
    }
    stage('npm run test'){
      steps{
        sh "npm run test"
      }
    }
    stage('npm build'){
      steps{
        sh "npm run build"
      }
    }
    stage('docker build'){
      environment {
        IMAGE = ${params.IMAGE_REPO_NAME}
        COMMIT_TAG = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        VERSION_TAG = readJSON( file: 'package.json').version
        BUILD_IMAGE_REPO_TAG = "${params.IMAGE_REPO_NAME}:${env.BUILD_TAG}"
      }
      steps{
        sh "docker build . -t $BUILD_IMAGE_REPO_TAG"
        sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:$COMMIT_TAG"
        sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:$VERSION_TAG"
        sh "docker tag $BUILD_IMAGE_REPO_TAG ${params.IMAGE_REPO_NAME}:${params.LATEST_BUILD_TAG}"
      }
    }
    stage('Remove Previous Stack'){
      when{
        return params.DOCKER_STACK_RM
      }
      steps{
        sh "docker stack rm ${params.DOCKER_STACK_NAME}"
      }
    }
    stage('Docker Stack Deploy'){
      steps{
        sh "docker deploy -c ${params.DOCKER_COMPOSE_FILENAME} ${params.DOCKER_STACK_NAME}"
      }
    }
  }
  post {
    always {
      sh 'echo "This will always run"'
    }
  }
}