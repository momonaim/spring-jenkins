pipeline {
        triggers {
            pollSCM('H/5 * * * *')
        }
    agent any

    environment {
        DOCKER_IMAGE = "docker.io/ita03mouadili/hello-jenkins"
        DOCKER_TAG = "latest"
    }

    tools {
        maven 'Maven'
        jdk 'JDK17'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/momonaim/spring-jenkins.git'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$DOCKER_TAG .'
            }
        }

        stage('Docker Login & Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push $DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('Deploy Kubernetes') {
            steps {
                withEnv([
                    "KUBECONFIG=/var/jenkins_home/.kube/config",
                    "MINIKUBE_HOME=/var/jenkins_home/.minikube",
                    "CLIENT_CERT=/var/jenkins_home/.minikube/profiles/minikube/client.crt",
                    "CLIENT_KEY=/var/jenkins_home/.minikube/profiles/minikube/client.key",
                    "CA_CERT=/var/jenkins_home/.minikube/ca.crt"
                ]) {
                    sh 'kubectl --client-certificate=$CLIENT_CERT --client-key=$CLIENT_KEY --certificate-authority=$CA_CERT config current-context'
                    sh 'kubectl --client-certificate=$CLIENT_CERT --client-key=$CLIENT_KEY --certificate-authority=$CA_CERT get nodes'
                    sh 'kubectl --client-certificate=$CLIENT_CERT --client-key=$CLIENT_KEY --certificate-authority=$CA_CERT apply -f k8s/'
                }
            }
        }
    }

    post {
        success {
            echo 'IMAGE PUSHED TO DOCKER HUB'
        }
    }
}
