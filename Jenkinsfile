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
        SONAR_TOKEN = credentials('sqa_397010f64a6ac64990d42ebbecfb285d2fe3cb4b') //make sure a right token is selected
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
                withSonarQubeEnv("${env.SONARQUBE_SERVER}") {
                    bat """
                        sonar-scanner ^
                        -Dsonar.projectKey=BMI_CL ^
                        -Dsonar.sources=src ^
                        -Dsonar.projectName=BMI_CL ^
                        -Dsonar.host.url=http://localhost:9000 ^
                        -Dsonar.login=${env.SONAR_TOKEN} ^
                        -Dsonar.java.binaries=target/classes
                    """
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
