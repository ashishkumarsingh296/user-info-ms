pipeline {
  agent any

  environment {
    DOCKER_CREDENTIAL = credentials('DOCKER-CREDENTIAL')
    VERSION = "${env.BUILD_ID}"
    IMAGE_NAME = 'ashishdevops1989/user-info-service'
    CONTAINER_NAME = 'user-info-service-container'
    PORT = '9096'
  }

  tools {
    maven "Maven"
  }

  stages {

    // Step 1: Checkout Latest Code
    stage('Checkout Code') {
      steps {
        git url: 'https://github.com/ashishkumarsingh296/user-info-ms.git', credentialsId: 'GITHUB-CREDS'
      }
    }

    // Step 2: Maven Build
    stage('Maven Build') {
      steps {
        script {
          bat 'mvn clean package -DskipTests' // Skip tests during package stage
        }
      }
    }

    // Step 3: Run Unit Tests
    stage('Run Tests') {
      steps {
        script {
          bat 'mvn test' // Run unit tests using Maven
        }
      }
    }

    // Step 4: Stop Running Docker Container (if exists)
    stage('Stop Docker Container') {
      steps {
        script {
          // Stop the container if it is already running (Windows)
          bat "docker stop ${CONTAINER_NAME} || exit /b 0"
        }
      }
    }

    // Step 5: Remove Docker Container (if exists)
    stage('Remove Docker Container') {
      steps {
        script {
          // Remove the container if it exists (Windows)
          bat "docker rm ${CONTAINER_NAME} || exit /b 0"
        }
      }
    }

    // Step 6: Remove Docker Image (if exists)
    stage('Remove Docker Image') {
      steps {
        script {
          // Remove the Docker image if it exists (if not already removed)
          bat "docker rmi ${IMAGE_NAME}:${VERSION} || exit /b 0"
        }
      }
    }

    // Step 7: Build Docker Image
    stage('Build Docker Image') {
      steps {
        script {
          docker.build("${IMAGE_NAME}:${VERSION}")
        }
      }
    }

    // Step 8: Push Docker Image
    stage('Push Docker Image') {
      steps {
        script {
          docker.withRegistry('https://index.docker.io/v1/', 'DOCKER-CREDENTIAL') {
            docker.image("${IMAGE_NAME}:${VERSION}").push()
          }
        }
      }
    }

    // Step 9: Run Docker Container
    stage('Run Docker Container') {
      steps {
        script {
          // Run the container with port mapping (9096)
          bat "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${IMAGE_NAME}:${VERSION}"
        }
      }
    }
  }

  post {
    // After the pipeline completes
    always {
      // Always run the following steps, no matter what
      echo 'Cleaning up workspace...'
      deleteDir()  // Clean up workspace after completion
    }

    success {
      // Runs only if the pipeline was successful
      echo 'Pipeline was successful. Image has been built and pushed to Docker Hub.'
    }

    failure {
      // Runs only if the pipeline failed
      echo 'Pipeline failed. Please check the logs for errors.'
      // You can add additional steps here to notify via email or Slack if needed
    }

    unstable {
      // Runs if the pipeline is unstable (e.g., tests failed)
      echo 'Pipeline is unstable. Some tests might have failed.'
    }

    aborted {
      // Runs if the pipeline was manually aborted
      echo 'Pipeline was aborted by the user.'
    }
  }
}
