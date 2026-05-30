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
                sh 'kubectl apply -f k8s/deployment.yaml --kubeconfig=/home/ceylin/.kube/config' // --validate=false, imagePullSecrets nedeniyle oluşabilecek hataları engeller
                sh 'kubectl apply -f k8s/service.yaml --kubeconfig=/home/ceylin/.kube/config'
                
                // ◄ YENİ SATIR: Kubernetes'e mevcut podları kapatıp Docker Hub'dan yenisini çekmeye zorlar
                sh 'kubectl rollout restart deployment/devops4-app --kubeconfig=/home/ceylin/.kube/config'
                
                // Dağıtımın durumunu terminale basıp kontrol edelim
                sh 'kubectl rollout status deployment/devops4-app --kubeconfig=/home/ceylin/.kube/config'

                // 3. ÖNEMLİ: Mevcut çalışan eski bir port-forward varsa çakışmasın diye önce onu sonlandırıyoruz
                sh 'pkill -f "port-forward" || true'
                
                // 4. Port yönlendirmeyi ARKA PLANDA (nohup ve & ile) başlatıyoruz
                // Böylece Jenkins komutu tetikler ve arkasına bakmadan pipeline'ı BAŞARIYLA bitirir.
                sh 'nohup kubectl port-forward service/devops4-service 8081:8081 --address 0.0.0.0 --kubeconfig=/home/ceylin/.kube/config > /dev/null 2>&1 &'
            }
        }
    }    
}          