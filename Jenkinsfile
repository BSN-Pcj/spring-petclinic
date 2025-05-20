pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK21"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerCredential')
        REGION = "ap-northeast-2"
        AWS_CREDENTIALS_NAME = "AWSCredentials"
    }
    
    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/BSN-Pcj/spring-petclinic.git',
                    branch: 'main'
            }
            post {
                success {
                    echo 'Git Clone Success'
                }
                failure {
                    echo 'Git Clone Fail'
                }
            }
        }
        // Maven build 작업
        stage('maven Build') {
            steps {
                echo 'maven Build'
                sh 'mvn -Dmaven.test.failure.ignore=true clean package' // Test error 무시
            }
        }

        // Docker Image 생성
        stage('Docker Image Build') {
            steps {
                echo 'Docker Image Build'
                dir("${env.WORKSPACE}") {
                   sh '''
                      docker build -t spring-petclinic:$BUILD_NUMBER .
                      docker tag spring-petclinic:$BUILD_NUMBER cj001214/spring-petclinic:latest
                      '''
                }
            }
        }

        // Docker Image Push
        stage('Docker Image Push') {
            steps {
                sh '''
                   echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                   docker push cj001214/spring-petclinic:latest
                   '''
            }
        }

        // Remove Docker Image
        stage('Remove Docker Image') {
            steps {
                sh '''
                docker rmi spring-petclinic:$BUILD_NUMBER
                docker rmi cj001214/spring-petclinic:latest
                '''
            }
        }
    }
}
