pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      parallel {
        stage('Compile') {
          steps {
            container('maven') {
              sh 'mvn compile'
            }
          }
        }
      }
    }
    stage('Test') {
      parallel {
        stage('Unit Tests') {
          steps {
            container('maven') {
              sh 'mvn test'
            }
          }
        }
      }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
        stage('Docker BnP') {
          steps {
            container('kaniko') {
              //sh '/kaniko/executor -f `pwd`/Dockerfile -c `pwd` --insecure --skip-tls-verify --cache=true --destination=docker.io/3788/dso-demo'
              sh '/kaniko/executor -f $(pwd)/Dockerfile -c $(pwd) --insecure --skip-tls-verify --cache=true --destination=docker.io/3788/dso-demo:latest'
            }
          }
        }
      }  // Closing 'parallel' block
    }  // Closing 'Package' stage
  }  // Closing 'stages' block
}  // Closing 'pipeline' block