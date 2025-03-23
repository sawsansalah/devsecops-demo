pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
    environment {
        container = "/kaniko/executor -f /home/jenkins/agent/workspace/Devsecops/Dockerfile -c /home/jenkins/agent/workspace/Devsecops --insecure --skip-tls-verify '--cache=true' '--destination=docker.io/3788/dso-demo:latest'"
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
    stage('SCA') {
      steps {
    container('maven') {
    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
    sh 'mvn org.owasp:dependency-check-maven:check'
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
              //sh '/kaniko/executor -f $(pwd)/Dockerfile -c $(pwd) --insecure --skip-tls-verify --cache=true --destination=docker.io/3788/dso-demo:latest'
              sh "${container}"

            }
          }
        }
      }  // Closing 'parallel' block
    }  // Closing 'Package' stage
  }  // Closing 'stages' block
}  // Closing 'pipeline' block