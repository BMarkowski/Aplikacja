pipeline {
    agent any

    environment {
        scannerHome = tool 'SonarQube'
        SONARQUBE_TOKEN = credentials('SONARQUBE_TOKEN')
        DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD')
        
    }

    stages {
        stage('Clear running apps') {
            steps {
                sh 'docker rm -f flask_app || true'
            }
        }
    
    stage('Sonarqube analysis frontend') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.login=${SONARQUBE_TOKEN}"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t flask_app:${BUILD_NUMBER} -t flask_app:latest ."
            }
        }
        stage('Run app') {
            steps {
                sh "docker run -d -p 127.0.0.1:5555:5555 --net=jenkins_docker_network --name flask_app -t flask_app:${BUILD_NUMBER}"
            }
        }

         stage('Selenium tests') {
            steps {
                dir('tests/') {
                    sh 'pip3 install -r requirements.txt'
                    sh 'python3 test_app.py'
                }
            }
        }
        stage('Upload Docker Image to Docker Hub') {
            steps {
                sh "docker login -u bmarkowskii -p ${DOCKER_HUB_PASSWORD}"
                sh "docker tag flask_app:${BUILD_NUMBER} bmarkowskii/flask_app:${BUILD_NUMBER}"
                sh 'docker tag flask_app:latest bmarkowskii/flask_app:latest'
                sh "docker push bmarkowskii/flask_app:${BUILD_NUMBER}"
                sh 'docker push bmarkowskii/flask_app:latest'
            }
        }
    }
}
