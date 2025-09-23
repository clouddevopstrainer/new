pipeline {
    agent any

    environment {
        APP_NAME = "spring-boot-app"
        IMAGE_NAME = "spring-boot-app:1.0"
        DOCKER_HUB_REPO = "your-dockerhub-username/spring-boot-app:1.0"

        MAVEN_HOME = tool 'Maven3'
        JAVA_HOME  = tool 'Java17'
        PATH = "${MAVEN_HOME}/bin:${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/your-repo/spring-boot-app.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker tag $IMAGE_NAME $DOCKER_HUB_REPO
                        docker push $DOCKER_HUB_REPO
                    '''
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                sh '''
                    sed -i "s|your-dockerhub-username/spring-boot-app:1.0|$DOCKER_HUB_REPO|g" k8s/deployment.yaml
                    kubectl apply -f k8s/deployment.yaml
                    kubectl apply -f k8s/service.yaml
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    kubectl get pods
                    kubectl get svc spring-boot-service
                '''
            }
        }
    }
}
