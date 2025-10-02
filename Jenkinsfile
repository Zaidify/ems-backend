pipeline {
    agent any

    tools {
        jdk 'Java'    // JDK configured in Jenkins
        maven 'Maven' // Maven configured in Jenkins
    }

    environment {
        EC2_USER = 'ubuntu'
        EC2_HOST = '13.232.111.201'
        SSH_CREDENTIALS = 'aws-ec2-key'
        APP_NAME = 'springboot-backend.jar'
        GIT_BASH = '"C:\\Program Files\\Git\\bin\\bash.exe"' // Path to Git Bash
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

//         stage('Test') {
//             steps {
//                 echo 'Running unit tests...'
//                 bat 'mvn test'
//             }
//         }

        stage('Package') {
            steps {
                echo 'Packaging Spring Boot application...'
                bat 'mvn package -DskipTests'
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                echo 'Deploying application to EC2...'
                sshagent([env.SSH_CREDENTIALS]) {
                    bat """
                    ${GIT_BASH} -c "scp -o StrictHostKeyChecking=no target\\*.jar ${EC2_USER}@${EC2_HOST}:/home/${EC2_USER}/${APP_NAME}"
                    ${GIT_BASH} -c "ssh ${EC2_USER}@${EC2_HOST} 'pkill -f ${APP_NAME} || true'"
                    ${GIT_BASH} -c "ssh ${EC2_USER}@${EC2_HOST} 'nohup java -jar /home/${EC2_USER}/${APP_NAME} > app.log 2>&1 &'"
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