// pipeline {
//   agent any

//   environment {
//     DOCKER_CREDENTIAL = credentials('DOCKER-CREDENTIAL')
//     VERSION = "${env.BUILD_ID}"
//     IMAGE_NAME = 'ashishdevops1989/user-info-service'
//     CONTAINER_NAME = 'user-info-service-container'
//     PORT = '9096'
//   }

//   tools {
//     maven "Maven"
//   }

//   stages {

//     // Step 1: Checkout Latest Code
//     stage('Checkout Code') {
//       steps {
//         git url: 'https://github.com/ashishkumarsingh296/user-info-ms.git', credentialsId: 'GITHUB-CREDS'
//       }
//     }

//     // Step 2: Maven Build
//     stage('Maven Build') {
//       steps {
//         script {
//           bat 'mvn clean package -DskipTests' // Skip tests during package stage
//         }
//       }
//     }

//     // Step 3: Run Unit Tests
//     stage('Run Tests') {
//       steps {
//         script {
//           bat 'mvn test' // Run unit tests using Maven
//         }
//       }
//     }

//     // Step 4: Stop Running Docker Container (if exists)
//     stage('Stop Docker Container') {
//       steps {
//         script {
//           // Stop the container if it is already running (Windows)
//           bat "docker stop ${CONTAINER_NAME} || exit /b 0"
//         }
//       }
//     }

//     // Step 5: Remove Docker Container (if exists)
//     stage('Remove Docker Container') {
//       steps {
//         script {
//           // Remove the container if it exists (Windows)
//           bat "docker rm ${CONTAINER_NAME} || exit /b 0"
//         }
//       }
//     }

//     // Step 6: Remove Docker Image (if exists)
//     stage('Remove Docker Image') {
//       steps {
//         script {
//           // Remove the Docker image if it exists (if not already removed)
//           bat "docker rmi ${IMAGE_NAME}:${VERSION} || exit /b 0"
//         }
//       }
//     }

//     // Step 7: Build Docker Image
//     stage('Build Docker Image') {
//       steps {
//         script {
//           docker.build("${IMAGE_NAME}:${VERSION}")
//         }
//       }
//     }

//     // Step 8: Push Docker Image
//     stage('Push Docker Image') {
//       steps {
//         script {
//           docker.withRegistry('https://index.docker.io/v1/', 'DOCKER-CREDENTIAL') {
//             docker.image("${IMAGE_NAME}:${VERSION}").push()
//           }
//         }
//       }
//     }

//     // Step 9: Run Docker Container
//     // stage('Run Docker Container') {
//     //   steps {
//     //     script {
//     //       // Run the container with port mapping (9096)
//     //       bat "docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${IMAGE_NAME}:${VERSION}"
//     //     }
//     //   }
//     // }
//   }

//   post {
//     // After the pipeline completes
//     always {
//       // Always run the following steps, no matter what
//       echo 'Cleaning up workspace...'
//       deleteDir()  // Clean up workspace after completion
//     }

//     success {
//       // Runs only if the pipeline was successful
//       echo 'Pipeline was successful. Image has been built and pushed to Docker Hub.'
//     }

//     failure {
//       // Runs only if the pipeline failed
//       echo 'Pipeline failed. Please check the logs for errors.'
//       // You can add additional steps here to notify via email or Slack if needed
//     }

//     unstable {
//       // Runs if the pipeline is unstable (e.g., tests failed)
//       echo 'Pipeline is unstable. Some tests might have failed.'
//     }

//     aborted {
//       // Runs if the pipeline was manually aborted
//       echo 'Pipeline was aborted by the user.'
//     }
//   }
// }



pipeline {
    agent any

    environment {
        DOCKER_CREDENTIAL = credentials('DOCKER-CREDENTIAL')
        VERSION = "${env.BUILD_ID}"
        IMAGE_NAME = 'ashishdevops1989/user-info-service'
        CONTAINER_NAME = 'user-info-service-container'
        MYSQL_CONTAINER_NAME = 'mysql-container'
        PORT = '9096'
        MYSQL_ROOT_PASSWORD = 'root'
        MYSQL_DATABASE = 'userdb'
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

        // Step 4: Start PostgreSQL container using Docker Compose (if not running) 
stage('Start PostgreSQL Container') {
    steps {
        script {
            echo "Starting PostgreSQL container using Docker Compose..."
            // Pull the latest PostgreSQL image and start the container if it's not already running
            bat """
            docker-compose -f docker-compose.yml up -d postgres
            """
        }
    }
}
        
        // // Step 4: Start MySQL container using Docker Compose (if not running)
        // stage('Start MySQL Container') {
        //     steps {
        //         script {
        //             echo "Starting MySQL container using Docker Compose..."
        //             // Pull the latest MySQL image and start the container if it's not already running
        //             bat """
        //             docker-compose -f docker-compose.yml up -d mysql
        //             """
        //         }
        //     }
        // }

        // Step 5: Stop Running Docker Container (if exists)
        stage('Stop Docker Container') {
            steps {
                script {
                    // Stop the user service container if it is already running (Windows)
                    bat "docker stop ${CONTAINER_NAME} || exit /b 0"
                }
            }
        }

        // Step 6: Remove Docker Container (if exists)
        stage('Remove Docker Container') {
            steps {
                script {
                    // Remove the user service container if it exists (Windows)
                    bat "docker rm ${CONTAINER_NAME} || exit /b 0"
                }
            }
        }

        // Step 7: Remove Docker Image (if exists)
        stage('Remove Docker Image') {
            steps {
                script {
                    // Remove the Docker image if it exists (if not already removed)
                    bat "docker rmi ${IMAGE_NAME}:${VERSION} || exit /b 0"
                }
            }
        }

        // Step 8: Build Docker Image
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${VERSION}")
                }
            }
        }

        // Step 9: Push Docker Image
        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'DOCKER-CREDENTIAL') {
                        docker.image("${IMAGE_NAME}:${VERSION}").push()
                    }
                }
            }
        }

        // Step 10: Run Docker Container
         stage('Run Docker Container') {
    steps {
        script {
            // Set the custom network and PostgreSQL container name
            def networkName = "mynetwork"
            def postgresContainerName = "postgres-container"
            
            // Run the user-info-service container and connect it to the custom network
            bat """
            docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} --network ${networkName} ${IMAGE_NAME}:${VERSION}
            """
        }
    }
}

        
        // stage('Run Docker Container') {
        //     steps {
        //         script {
        //             // Run the user-info-service container and connect it to MySQL
        //             bat """
        //             docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} --link ${MYSQL_CONTAINER_NAME}:mysql ${IMAGE_NAME}:${VERSION}
        //             """
        //         }
        //     }
        // }

        // Step 11: Cleanup Docker Containers
        stage('Cleanup Docker Containers') {
            steps {
                script {
                    // Stop and remove the MySQL container and the user service container after the job
                    bat """
                    docker-compose -f docker-compose.yml down
                    """
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

