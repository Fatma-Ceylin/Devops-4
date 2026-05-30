pipeline {
    agent any

    environment {
        // Docker Hub bilgilerinizi buraya yazın
        DOCKER_HUB_USER = 'fceylin'
        IMAGE_NAME      = 'forth'
        REGISTRY_CRED   = 'dockerhub-credentials' // Bunu Jenkins arayüzünde oluşturacağız
    }

    stages {
        // Stage 1: Clone the project from Github (Jenkins bunu otomatik de yapabilir ama tanımlamak iyidir)
        stage('Clone') {
            steps {
                checkout scm
            }
        }

        // Stage 2: Build the project and create a jar file
        stage('Gradle Build') {
            steps {
                sh 'chmod +x gradlew'
                sh './gradlew bootJar'
            }
        }

        // Stage 3, 4 & 5: Docker Build, Login and Push
        stage('Docker Build & Push') {
            steps {
                script {
                    // Jenkins'in Docker Hub şifrenizi güvenli okuması için credential kullanıyoruz
                    withCredentials([usernamePassword(credentialsId: "${REGISTRY_CRED}", usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        // Stage 3: Create a docker image
                        sh "docker build -t ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest ."
                        
                        // Stage 4: Login to the Dockerhub
                        sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                        
                        // Stage 5: Push the image to the hub
                        sh "docker push ${DOCKER_HUB_USER}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        // Stage 6: Run the K8s deployment and service scripts
        stage('Deploy to K8s') {
            steps {
                // Minikube/Kubernetes konfigürasyonunu Jenkins kullanıcısının görebileceği şekilde çalıştırıyoruz
                sh 'env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy kubectl apply -f k8s/deployment.yaml ' // --validate=false, imagePullSecrets nedeniyle oluşabilecek hataları engeller
                sh 'env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxy kubectl apply -f k8s/service.yaml'
                
                // Dağıtımın durumunu terminale basıp kontrol edelim
                sh 'env -u HTTP_PROXY -u HTTPS_PROXY -u http_proxy -u https_proxykubectl rollout status deployment/devops4-app'
            }
        }
    }
}