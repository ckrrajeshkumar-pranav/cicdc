pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'ckrrajeshkumar/train-schedule'
        KUBE_NAMESPACE = 'default'  // Change if using a different namespace
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/ckrrajeshkumar-pranav/cicdc.git', credentialsId: 'github-credentials'
            }
        }
        stage('Build Application') {
            steps {
                sh './gradlew build --no-daemon'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    def commitHash = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.DOCKER_TAG = "${commitHash}"
                }
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG -t $DOCKER_IMAGE:latest .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE:$DOCKER_TAG'
                    sh 'docker push $DOCKER_IMAGE:latest'
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'kube', variable: 'KUBECONFIG')]) {
                    script {
                        sh """
                        kubectl --kubeconfig $KUBECONFIG --insecure-skip-tls-verify apply -f k8s/deployment.yaml
                        kubectl --kubeconfig $KUBECONFIG --insecure-skip-tls-verify apply -f k8s/service.yaml
                        kubectl --kubeconfig $KUBECONFIG --insecure-skip-tls-verify apply set image deployment/train-schedule train-schedule=$DOCKER_IMAGE:$DOCKER_TAG
                        kubectl --kubeconfig $KUBECONFIG --insecure-skip-tls-verify apply rollout status deployment/train-schedule -n $KUBE_NAMESPACE || exit 1
                        """
                    }
                }
            }
        }
    }
}

