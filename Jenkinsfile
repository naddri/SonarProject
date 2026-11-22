pipeline {
    agent any

    tools {
        maven 'maven'   // Must match the name configured in Jenkins Global Tool Configuration
    }

    environment {
        // these are the environment variables (Amir's machine) that will be used in the pipeline
       PATH = "C:\\Program Files\\Docker\\Docker\\resources\\bin;${env.PATH}"
        JAVA_HOME = 'C:/Program Files/Java/jdk-27'


        SONARQUBE_SERVER = 'XXXXX'
        SONAR_TOKEN = credentials('XXXXX') //make sure a right token is selected
        DOCKERHUB_CREDENTIALS_ID = 'XXXXX'
        DOCKERHUB_REPO = 'XXXX/bmi_3013_OMANIMI'
        DOCKER_IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/naddri/SonarProject'
            }
        }

        stage('Build') {
            steps {
                bat 'mvn clean install'
            }
        }

        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }
// Check the SonarQube analysis stage for the correct project key and name
        stage('SonarQube Analysis') {
    steps {
        withCredentials([string(
            credentialsId: 'SONAR_TOKEN',
            variable: 'SONAR_TOKEN'
        )]) {
            bat '''
                mvn sonar:sonar ^
                -Dsonar.projectKey=demo1 ^
                -Dsonar.host.url=http://localhost:9000 ^
                -Dsonar.token=%SONAR_TOKEN%
            '''
        }
    }
}

        stage('Build Docker Image') {
            steps {
                script {
                    def app = docker.build("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}")
                    env.BUILT_IMAGE = app.id  // Optional: store image ID for reuse
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKERHUB_CREDENTIALS_ID) {
                       docker.image("${DOCKERHUB_REPO}:${DOCKER_IMAGE_TAG}").push()
                    }
                }
            }
        }
    }


}
