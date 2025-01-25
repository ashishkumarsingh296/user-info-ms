pipeline {
    agent any

    environment {
        DOCKER_CREDENTIAL = credentials('DOCKER-CREDENTIAL')
        VERSION = "${env.BUILD_ID}"
        IMAGE_NAME = 'ashishdevops1989/user-info-service'
        CONTAINER_NAME = 'user-info-service-container'
        POSTGRES_CONTAINER_NAME = 'postgres-container'
        PORT = '9096'
        POSTGRES_DB = 'postgres'
        POSTGRES_USER = 'postgres'
        POSTGRES_PASSWORD = 'root'
        NETWORK_NAME = 'mypostgresnetwork' // Custom network for services
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

        // Step 4: Run PostgreSQL Container (Ensure it is using the correct network)
     stage('Run PostgreSQL Container') {
    steps {
        script {
            echo "Checking if postgres-container exists..."

            // Check if the container exists
            def containerExists = bat(script: "docker ps -aq -f name=postgres-container", returnStdout: true).trim()

            // If the container doesn't exist, create and start it
            if (containerExists.isEmpty()) {
                echo "postgres-container does not exist. Creating and starting a new one..."

                bat """
                docker run -d --name postgres-container --network ${NETWORK_NAME} -e POSTGRES_PASSWORD=${POSTGRES_PASSWORD} -e POSTGRES_DB=${POSTGRES_DB} -e POSTGRES_USER=${POSTGRES_USER} postgres
                """
            } else {
                echo "postgres-container already exists and is running."
            }
        }
    }
}


        // Step 5: Stop and Remove Existing Docker Containers (if necessary)
        stage('Stop & Remove Docker Containers') {
            steps {
                script {
                    // Ensure the Spring Boot container is stopped and removed
                    bat "docker stop ${CONTAINER_NAME} || exit /b 0"
                    bat "docker rm ${CONTAINER_NAME} || exit /b 0"
                }
            }
        }

        // Step 6: Build Docker Image
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${VERSION}")
                }
            }
        }

        // Step 7: Push Docker Image
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'DOCKER-CREDENTIAL') {
                        docker.image("${IMAGE_NAME}:${VERSION}").push()
                    }
                }
            }
        }

stage('Run Docker Container') {
    steps {
        script {
            echo "Waiting for PostgreSQL container to be ready..."
            bat "timeout /t 30" // Wait for 30 seconds
            echo "Running user-info-service container with PostgreSQL..."
            
            // Run the Spring Boot container
            bat """
            docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} --network ${NETWORK_NAME} ${IMAGE_NAME}:${VERSION}
            """
        }
    }
   }
   }

    post {
        always {
            // Clean up workspace after completion
            echo 'Cleaning up workspace...'
            deleteDir()
        }

        success {
            echo 'Pipeline was successful. Image has been built and pushed to Docker Hub.'
        }

        failure {
            echo 'Pipeline failed. Please check the logs for errors.'
        }

        unstable {
            echo 'Pipeline is unstable. Some tests might have failed.'
        }

        aborted {
            echo 'Pipeline was aborted by the user.'
        }
    }
}
