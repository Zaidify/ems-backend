pipeline {
    agent any

    tools {
        jdk 'Java'
        maven 'Maven'
    }

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '13.232.111.201'
        APP_NAME = 'springboot-backend-0.0.1-SNAPSHOT.jar'
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Cloning Git repository...'
                git branch: 'Main', url: 'https://github.com/Zaidify/ems-backend'
            }
        }

        stage('Build') {
            steps {
                echo 'Building project with Maven...'
                bat 'mvn clean install -DskipTests'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging Spring Boot application...'
                bat 'mvn package -DskipTests'
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                echo "Deploying application to EC2..."
                sshagent(['aws-ec2-key']) {  // Jenkins SSH credential ID
                    bat """
                        REM Copy jar to EC2 (ignore host key verification)
                        scp -o StrictHostKeyChecking=no target\\${APP_NAME} %EC2_USER%@%EC2_HOST%:/home/ubuntu/

                        REM Run Spring Boot app on EC2
                        ssh -o StrictHostKeyChecking=no %EC2_USER%@%EC2_HOST% "nohup java -jar /home/ubuntu/${APP_NAME} > /home/ubuntu/app.log 2>&1 &"
                    """
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline finished successfully!'
        }
        failure {
            echo 'Pipeline failed. Check Jenkins logs!'
        }
    }
}
