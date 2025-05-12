pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mohitv47/demo-app'
        DOCKER_REGISTRY = 'https://index.docker.io'  // DockerHub registry URL
    }

    triggers {
        githubPush()  // Trigger build on GitHub push to master branch
    }

    stages {
        stage('Clone Repo') {
            steps {
                // Clone the repository from GitHub
                git branch: 'master', url: 'https://github.com/Mohitv47/demo-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image
                    sh 'docker build -t $DOCKER_IMAGE:latest .'
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                // Login to Docker Hub and push the image
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        // Login to Docker Hub using the credentials
                        sh """
                            echo $DOCKER_PASS | docker login $DOCKER_REGISTRY -u $DOCKER_USER --password-stdin
                            docker push $DOCKER_IMAGE:latest
                        """
                    }
                }
            }
        }

        stage('Deploy Container') {
            steps {
                script {
                    // Stop and remove any existing container named demo-app, then run the new one
                    sh 'docker rm -f demo-app || true'
                    sh 'docker pull $DOCKER_IMAGE:latest'
                    sh 'docker run -d -p 3000:3000 --name demo-app $DOCKER_IMAGE:latest'
                }
            }
        }
    }

    post {
        success {
            emailext(
                to: 'ananda.yashaswi@quokkalabs.com,prateek.roy@quokkalabs.com',
                subject: "✅ Build SUCCESS - ${env.JOB_NAME}",
                body: """\
The Jenkins pipeline for ${env.JOB_NAME} completed successfully.

Build URL: ${env.BUILD_URL}
"""
            )
        }

        failure {
            emailext(
                to: 'ananda.yashaswi@quokkalabs.com,prateek.roy@quokkalabs.com',
                subject: "❌ Build FAILED - ${env.JOB_NAME}",
                body: """\
The Jenkins pipeline for ${env.JOB_NAME} failed.

Build URL: ${env.BUILD_URL}
Check the console output for more details.
"""
            )
        }
    }
}
