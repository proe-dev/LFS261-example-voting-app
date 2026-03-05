pipeline {
  agent none

  stages {
    stage ("hello"){
      steps {
        echo "Hello world"
      }
    }
    stage ("worker-build"){
      when {
        changeset "**/worker/**"
      }
      agent {
        docker {
          image 'maven:3.9.8-sapmachine-21'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'Compiling worker app..'
        dir('worker') {
          sh 'mvn compile'
        }
      }
    }
    stage ("worker-test") {
      when {
        changeset "**/worker/**"
      }
      agent {
        docker {
          image 'maven:3.9.8-sapmachine-21'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'Running Unit Tests on worker app..'
        dir('worker') {
          sh 'mvn clean test'
        }
      }
    }
    stage ("worker-package") {
      when {
        changeset "**/worker/**"
        branch 'master'
      }
      agent {
        docker {
          image 'maven:3.9.8-sapmachine-21'
          args '-v $HOME/.m2:/root/.m2'
        }
      }
      steps {
        echo 'Packaging worker app'
        dir('worker') {
          sh 'mvn package -DskipTests'
          archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
        }
      }
    }
    stage ("worker-docker-package") {
      when {
        changeset "**/worker/**"
      //  branch 'master'
      }
      agent any
      steps {
        echo 'Packaging worker app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def workerImage = docker.build("meapeze/worker:v${env.BUILD_ID}", "./worker")
            workerImage.push()
            workerImage.push("latest")
          }
        }
      }
    }
    stage('vote-build'){ 
      agent{
        docker{
          image 'python:3.11-slim'
          args '--user root'
        }
      }
      steps{ 
        echo 'Compiling vote app.' 
        dir('vote'){
          sh "pip install -r requirements.txt"
        } 
      } 
    } 
    stage('vote-test'){ 
      agent {
        docker{
          image 'python:3.11-slim'
          args '--user root'
        }
      }
      steps{ 
        echo 'Running Unit Tests on vote app.' 
        dir('vote'){ 
          sh "pip install -r requirements.txt"
          sh 'nosetests -v'
        } 
      } 
    }
    stage('vote-docker-package'){
      agent any
      steps{
        echo 'Packaging vote app with docker'
        script{
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            // ./vote is the path to the Dockerfile that Jenkins will find from the Github repo
            def voteImage = docker.build("meapeze/vote:v${env.BUILD_ID}", "./vote")
            voteImage.push()
            voteImage.push("latest")
          }
        }
      }
    }
    stage ("result-build"){
      when {
        changeset "**/result/**"
      }
      steps {
        echo 'Compiling result app..'
        dir('result') {
          sh 'npm install'
        }
      }
    }
    stage ("result-test") {
      when {
        changeset "**/result/**"
      }
      steps {
        echo 'Running Unit Tests on result app..'
        dir('result') {
          sh 'npm install'
          sh 'npm test'
        }
      }
    }
    stage ("result-docker-package") {
      when {
        changeset "**/result/**"
      //  branch 'master'
      }
      agent any
      steps {
        echo 'Packaging result app with docker'
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
            def resultImage = docker.build("meapeze/result:v${env.BUILD_ID}", "./result")
            resultImage.push()
            resultImage.push("latest")
          }
        }
      }
    }
  }
  post {
    always {
      echo 'Building multibranch pipeline for instavote stack is completed..'
    }
  }
}
