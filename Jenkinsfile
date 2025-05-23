pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = credentials('dockerHub') 
        IMAGE_TAG = "${BUILD_NUMBER}"       
        DOCKER_IMAGE = "taharamakda/chatapp"          
    }

  stages {
      
        stage('Cleanup Workspace') {
            steps {
                deleteDir() 
            }
        }
        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
                sh 'git clone -b main https://github.com/TahaRamkda/chatApp.git'
            }
        }
        stage('Check Workspace Files') {
            steps {
                sh 'ls -la'
                sh 'ls -la chatApp'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQubeServer') {
                sh 'sonar-scanner'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building the Docker image...'
                sh 'docker build -t taharamakda/chatapp:latest -f chatApp/Dockerfile chatApp'
            }
        }

        stage('Image Scan (Trivy)') {
        steps {
            sh "trivy image --scanners config --format table -o report.txt $DOCKER_IMAGE:$IMAGE_TAG || true"  
        }
        }

        stage('Push to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHub') {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }
        stage('Clone Repository') {
            steps {
                echo 'Cloning the repository...'
                sh 'git clone -b main https://github.com/TahaRamkda/menifest.git'
            }
        }

        stage('Update Image Tag in Deployment YAML') {
            steps {
                sh "sed -i '' 's|image: $DOCKER_IMAGE:.*|image: $DOCKER_IMAGE:$IMAGE_TAG|' deploy.yml"
                sh 'git config user.name "jenkins"'
                sh 'git config user.email "jenkins@ci.local"'
                sh 'git commit -am "Update image tag to $IMAGE_TAG"'
                sh 'git push origin main'
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}

