pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }

  environment {
    // Define the Kaniko command as an environment variable
    KANIKO_COMMAND = "/kaniko/executor -f /home/jenkins/agent/workspace/Devsecops/Dockerfile -c /home/jenkins/agent/workspace/Devsecops --insecure --skip-tls-verify --cache=true --destination=docker.io/3788/dso-demo:latest"
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

    stage('Testing') {
      parallel {
        stage('Test') {
          steps {
            container('maven') {
              sh 'mvn test'
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

      post {
        always {
          archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true, onlyIfSuccessful: true
          // Uncomment the following line if you want to publish the dependency check report
          // dependencyCheckPublisher pattern: 'target/dependency-check-report.html'
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

        stage('Docker Build & Push') {
          steps {
            container('kaniko') {
              // Use the environment variable for the Kaniko command
              sh "${KANIKO_COMMAND}"
            }
          }
        }
      } // Closing 'parallel' block
    } // Closing 'Package' stage
  } // Closing 'stages' block
} // Closing 'pipeline' block