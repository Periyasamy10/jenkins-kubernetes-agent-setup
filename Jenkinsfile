pipeline {
  agent {
    kubernetes {
      label 'jenkins-slave'
      defaultContainer 'gradle'
      podRetention idle(10)
      yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: jnlp
    image: jenkins/inbound-agent:jdk17
  - name: gradle
    image: gradle:8.4-jdk21
    command:
    - cat
    tty: true
"""
    }
  }
  stages {
    stage('Test Agent') {
      steps {
        echo 'Running on custom container...'
        sh 'gradle --version'
      }
    }
  }
}
