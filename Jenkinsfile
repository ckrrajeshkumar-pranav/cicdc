pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ckrrajeshkumar/train-schedule'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ckrrajeshkumar-pranav/cicdc.git', credentialsId: 'github-credentials'
            }
        } 
        stage('Set Executable Permission') {
            steps {
                script {
                    sh 'chmod +x gradlew'  // Ensure gradlew is executable
                }
            }
        } 
        stage('List Files') {
            steps {
                script {
                    sh 'ls -l'  // Confirm gradlew file is present and executable
                }
            }
        } 
        stage('Build Application') {
            steps {
                script {
                    sh './gradlew build --no-daemon'
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.DOCKER_TAG = "${commitHash}"
                }
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'
                }
            }
        } 
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kuber', variable: 'KUBECONFIG')]) {  // Use the kubeconfig credentials
                    script {
                        sh """
                        kubectl --kubeconfig $KUBECONFIG  set image deployment/train-schedule train-schedule=$DOCKER_IMAGE:$DOCKER_TAG
                        kubectl --kubeconfig $KUBECONFIG  rollout status deployment/train-schedule
                        """
                    }
                }
            }
        }
    }
}
