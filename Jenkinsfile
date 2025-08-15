pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "unname-app:${env.BUILD_NUMBER}"
        REMOTE_SERVER = "31.129.49.244"
        SSH_CREDS = "server-ssh-creds"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm: [
                    $class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/Sorok3Ro/unname.git',
                        credentialsId: 'github-creds'
                    ]]
                ]
            }
        }
        stage('Test Docker') {
             steps {
                     bat '"C:\\Program Files\\Docker\\Docker\\Resources\\bin\\docker.exe" --version'
                     bat '"C:\\Program Files\\Docker\\Docker\\Resources\\bin\\docker.exe" images'
    }
}
        stage('Build Docker Image') {
            steps {
                script {
                    // Явное указание пути к Docker для Windows
                    bat "\"${env.DOCKER_PATH}\" build -t ${env.DOCKER_IMAGE} ."
                }
            }
        }
        stage('Deploy to Server') {
            steps {
                script {
                    // Сохраняем Docker-образ в .tar
                    bat '"C:\\Program Files\\Docker\\Docker\\Resources\\bin\\docker.exe" save -o image.tar ${DOCKER_IMAGE}'
                    
                    // Копируем образ на сервер
                    sshPut(
                        remote: REMOTE_SERVER,
                        credentialsId: SSH_CREDS,
                        from: 'image.tar',
                        into: '/tmp/'
                    )
                    
                    // Загружаем образ и запускаем контейнер
                    sshCommand(
                        remote: REMOTE_SERVER,
                        credentialsId: SSH_CREDS,
                        command: """
                            docker load -i /tmp/image.tar && \
                            docker stop unname-container || true && \
                            docker rm unname-container || true && \
                            docker run -d --name unname-container -p 8080:80 ${DOCKER_IMAGE} 
                        """
                    )
                }
            }
        }
    }
    post {
        always {
            // Очистка временных файлов для Windows
            bat 'del /f image.tar 2>nul || echo No image.tar to delete'
        }
    }
}