pipeline {
    agent any

    environment {
        // Используем секреты и переменные, которые мы настроили ранее
        DOCKER_CREDS = 'dockerhub-creds'
        KUBE_CREDS   = 'kubernetes-creds'
        
        // Укажи здесь свой логин Docker Hub
        DOCKER_USER  = 'akishevakzhan@gmail.com' 
        APP_NAME     = 'web-msvc-app'
    }

    stages {
        stage('Checkout') {
            steps {
                // Скачивание кода из GitHub
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                // Имитация сборки (например, npm install или mvn package)
                echo "Building microservice..."
                sh 'ls -R' 
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    // Сборка образа и отправка в Docker Hub
                    docker.withRegistry('', "${DOCKER_CREDS}") {
                        def customImage = docker.build("${DOCKER_USER}/${APP_NAME}:${env.BUILD_NUMBER}")
                        customImage.push()
                        customImage.push("latest")
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                // Деплой через Helm с использованием наших переменных окружения
                sh """
                helm upgrade --install ${env.HELM_PROJECT} ${env.HELM_CHART} \
                --set image.repository=${DOCKER_USER}/${APP_NAME} \
                --set image.tag=${env.BUILD_NUMBER} \
                --namespace ${env.CLUSTER_NAMESPACE}
                """
            }
        }
    }

    post {
        success {
            echo "Successfully deployed version ${env.BUILD_NUMBER}"
        }
        failure {
            echo "Build failed. Check the logs in Blue Ocean."
        }
    }
}
