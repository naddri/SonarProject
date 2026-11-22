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
        DOCKERHUB_CREDENTIALS_ID = 'naddri'
        DOCKERHUB_REPO = 'XXXX/bmi_3013_rafi'
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
                    bat '''
                        mvn org.sonarsource.scanner.maven:sonar-maven-plugin:sonar ^
                        -Dsonar.projectKey=PROJECTNUPPI ^
                        -Dsonar.projectName=BMI_CL ^
                        -Dsonar.token=%SONAR_TOKEN%
                    '''
                }
            }
        }

        


stage('Build Docker Image') {
    steps {
        bat '''
            @echo off
            set "DOCKER=C:\\Users\\nadri\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe"

            if not exist "%DOCKER%" (
                echo Docker.exe ei loydy
                exit /b 1
            )

            "%DOCKER%" --version
            if errorlevel 1 exit /b 1

            "%DOCKER%" info
            if errorlevel 1 exit /b 1

            "%DOCKER%" build -t "%DOCKERHUB_REPO%:%DOCKER_IMAGE_TAG%" .
            if errorlevel 1 exit /b 1
        '''
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
