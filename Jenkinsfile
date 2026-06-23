pipeline {
    agent any

    environment {
        // ⚠️ तुमचा अचूक Docker Hub युझरनेम इथे टाका
        DOCKER_HUB_USER = 'udayyadav22' 
        IMAGE_NAME      = 'todo-app'
        IMAGE_TAG       = "${env.BUILD_NUMBER}"
        REGISTRY_CRED   = 'docker-hub-credentials'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'
                script {
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                echo 'Pushing Image to Docker Hub...'
                withCredentials([usernamePassword(credentialsId: env.REGISTRY_CRED, passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Deploy to KIND Kubernetes') {
            steps {
                echo 'Deploying to KIND Cluster...'
                script {
                    // १. इमेज टॅग डायनॅमिकली फाईलमध्ये अपडेट करणे
                    sh "sed -i 's|udayyadav22/todo-app:[0-9]*|${DOCKER_HUB_USER}/${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/todo-k8s.yaml || true"
                    
                    // २. KIND वर डिप्लॉय करणे
                    sh "kubectl apply -f k8s/todo-k8s.yaml"
                    
                    // ३. रोलआऊट स्टेटस तपासणे
                    sh "kubectl rollout status deployment/todo-app-deployment || echo 'Deployment triggered'"
                }
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline यशस्वीरित्या पूर्ण झाली! 🎉'
            sh "docker logout"
        }
        failure {
            echo 'Pipeline फेल झाली. कृपया लॉग्स तपासा! ❌'
            sh "docker logout"
        }
    }
}
