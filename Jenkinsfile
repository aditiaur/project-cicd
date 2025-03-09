pipeline {	
	agent {
    docker {
        image 'maven:3.8.6-openjdk-11'
        args '--user root -v /var/run/docker.sock:/var/run/docker.sock'
    }
}

    stages {
        stage('Checkout') {
            steps {
                sh 'echo passed'
                // Example: git branch: 'main', url: 'https://github.com/aditiaur/project-cicd.git'
            }
        }
        stage('Build and Test') {
            steps {
                sh 'ls -ltr'
                sh 'cd spring-boot-app && mvn clean package'
            }
        }
        stage('Static Code Analysis') {
            environment {
                SONAR_URL = "http://13.232.207.190:9000/"
            }
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_AUTH_TOKEN')]) {
                    sh 'cd spring-boot-app && mvn sonar:sonar -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=$SONAR_URL'
                }
            }
        }
        stage('Build and Push Docker Image') {
            environment {
                DOCKER_IMAGE = "aditiaur12/project-cicd:${BUILD_NUMBER}"
                REGISTRY_CREDENTIALS = credentials('dockerhub-creds')
            }
            steps {
                script {
                    sh 'cd spring-boot-app && docker build -t ${DOCKER_IMAGE} .'
                    def dockerImage = docker.image("${DOCKER_IMAGE}")
                    docker.withRegistry('https://index.docker.io/v1/', "dockerhub-creds") {
                        dockerImage.push()
                    }
                }
            }
        }
        stage('Update Deployment File') {
            environment {
                GIT_REPO_NAME = "project-cicd"
                GIT_USER_NAME = "aditiaur"
            }
            steps {
                withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                    git config --global user.email "aditiaur122000@gmail.com"
                    git config --global user.name "aditiaur"

                    git checkout main || git checkout -b main
                    git pull origin main

                    
					sh """
                sed -i 's#\\(image: aditiaur12/project-cicd:\\).*#\\1${BUILD_NUMBER}#' spring-boot-app-manifests/deployment.yml
                    """



                    cat spring-boot-app-manifests/deployment.yml

                    git add spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}" || echo "No changes to commit"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
            }
        }
    }
}
