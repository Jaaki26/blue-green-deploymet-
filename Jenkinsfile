pipeline {
    agent any
    tools {
        maven 'Maven3'
        jdk 'Java17'
    }
    environment {
        DOCKER_IMAGE = "your-dockerhub-username/bluegreen-sample"
        KUBE_NAMESPACE = "default"
    }
    stages {
        stage('Checkout') {
            steps { git 'https://github.com/your-fork/bluegreen-sample.git' }
        }
        stage('Build with Maven') {
            steps { sh 'mvn clean package -DskipTests' }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }
        stage('Trivy Scan') {
            steps { sh 'trivy fs --exit-code 0 --severity HIGH .' }
        }
        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
                sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
            }
        }
        stage('Deploy Blue-Green to EKS') {
            steps {
                sh 'kubectl apply -f k8s/deployment-blue.yaml'
                sh 'kubectl apply -f k8s/deployment-green.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }
    post {
        success {
            mail to: 'team@example.com',
                 subject: "Pipeline Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The build and deployment completed successfully."
        }
        failure {
            mail to: 'team@example.com',
                 subject: "Pipeline Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                 body: "The build or deployment failed. Please check Jenkins logs."
        }
    }
}
