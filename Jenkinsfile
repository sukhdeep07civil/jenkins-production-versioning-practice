pipeline {
    agent any

    environment {
        IMAGE = "ssm0712/jenkins-production-versioning-practice"
    }

    stages{
        stage('Checkout scm'){
            steps{
                checkout scm
            }
        }

        stage('Set Version'){
            steps{
                script{
                    env.SHORT_SHA = bat(
                        script : '@echo off && git rev-parse --short HEAD',
                        returnStdout:true
                    ).trim()

                    env.VERSION = "${BUILD_NUMBER}-${env.SHORT_SHA}"

                    echo "BUILD-VERSION : ${env.VERSION}"
                }
            }
        }

        stage('Build Image'){
            steps{
                bat """
                docker build -t %IMAGE%:%VERSION% .
                """
            }
        }

        stage('Tag images'){
            steps{
                bat """
                docker tag %IMAGE%:%VERSION% %IMAGE%:%BUILD_NUMBER%
                docker tag %IMAGE%:%VERSION% %IMAGE%:latest
                """
            }
        }

        stage('Push images'){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]){
                    bat """
                    docker login -u %DOCKER_USER% -p %DOCKER_PASS%
                    docker push %IMAGE%:%VERSION%
                    docker push %IMAGE%:%BUILD_NUMBER%
                    docker push %IMAGE%:latest
                    """
                }
            }
        }

        stage('Deploy with Docker compose'){
            steps{
                bat """
                docker-compose down || echo docker compose down
                docker-compose pull
                docker-compose up -d
                """
            }
        }
    }
}