pipeline {
    agent {
        label 'general'
    }

    triggers {
        githubPush()
    }

    options {
        timeout(time: 10, unit: 'MINUTES')  // Discard the build after 10 minutes of running
        timestamps()  // Display timestamp in console output
    }

    environment {
        IMAGE_TAG = "v1.0.$BUILD_NUMBER"
        IMAGE_BASE_NAME = "netflix-frontend-dev"

        DOCKER_CREDS = credentials('dockerhub')  // Ensure 'dockerhub' matches your Jenkins credential ID
        DOCKER_USERNAME = "${DOCKER_CREDS_USR}"  // Accessing the username from the credential
        DOCKER_PASS = "${DOCKER_CREDS_PSW}"      // Accessing the password from the credential
    }

    stages {
        stage('Docker setup') {
            steps {
                sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USERNAME --password-stdin
                '''
            }
        }

        stage('Build app container') {
            steps {
                script {
                    def imageFullName = "${env.DOCKER_USERNAME}/${env.IMAGE_BASE_NAME}:${env.IMAGE_TAG}"
                    sh '''
                        docker build -t $imageFullName .
                        docker push $imageFullName
                    '''
                }
            }
        }

        stage('Trigger Deploy') {
            steps {
                build job: 'NetflixDeployDev', wait: false, parameters: [
                    string(name: 'SERVICE_NAME', value: "NetflixFrontend"),
                    string(name: 'IMAGE_FULL_NAME_PARAM', value: "${DOCKER_USERNAME}/${IMAGE_BASE_NAME}:${IMAGE_TAG}")
                ]
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
