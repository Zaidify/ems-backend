pipeline {
    agent any

    tools {
        jdk 'Java'
        maven 'Maven'
    }

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '13.232.111.201'
        SSH_CREDENTIALS = 'aws-ec2-key'
        APP_NAME = 'springboot-backend.jar'
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
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh 'mvn test'
            }
        }

        stage('Package') {
            steps {
                echo 'Packaging Spring Boot application...'
                sh 'mvn package -DskipTests'
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                echo 'Deploying application to EC2...'
                sshagent([env.SSH_CREDENTIALS]) {
                    sh """
                    scp -o StrictHostKeyChecking=no target/*.jar ${EC2_USER}@${EC2_HOST}:/home/${EC2_USER}/${APP_NAME}
                    ssh ${EC2_USER}@${EC2_HOST} 'pkill -f ${APP_NAME} || true'
                    ssh ${EC2_USER}@${EC2_HOST} 'nohup java -jar /home/${EC2_USER}/${APP_NAME} > app.log 2>&1 &'
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