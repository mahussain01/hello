pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        KUBECONFIG =  'C:\\Users\\ahamm\\.kube\\config'
        MINIKUBE_HOME = 'C:\\Users\\ahamm\\.minikube'
    }

    stages {
        
        stage('Debug') {
            steps {
                echo "KUBECONFIG path: ${KUBECONFIG}"
                echo "MINIKUBE_HOME path: ${MINIKUBE_HOME}"
            }
        }
        stage('Checkout') {
            steps {
                // Checkout your project from the source control
               //checkout([$class: 'GitSCM', userRemoteConfigs: [[url: 'https://github.com/mahussain01/hello']]])
               git credentialsId: 'git-creds', url: 'https://github.com/mahussain01/hello'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Build your Docker image
                script {
                    echo 'Buid Docker Image'
                    docker build -t mahussain01/hello:${BUILD_NUMBER} .

                    //docker.build("${DOCKER_IMAGE}")
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                // Push the Docker image to your registry
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-creds') {
                    echo 'Push to Repo'
                    docker push mahussain01/hello:${BUILD_NUMBER}

                    //docker.image("${DOCKER_IMAGE}").push()
                    }
                }
            }
        }

        stage('Deploy to Minikube') {
            steps {
                // Set the Docker environment to use Minikube's Docker daemon
                script {
                    withEnv(["DOCKER_HOST=tcp://127.0.0.1:50891", "DOCKER_CERT_PATH=${MINIKUBE_HOME}/.minikube/certs", "DOCKER_TLS_VERIFY=1"]) {
                        // Run your container in Minikube
                        cat deployment.yaml
                        sed -i '' "s/32/${BUILD_NUMBER}/g" deployment.yaml
                        cat deployment.yaml
                        git add deployment.yaml
                        git commit -m 'Updated the deploy yaml | Jenkins Pipeline'
                        git remote -v
                        git push https://github.com/mahussain01/hello HEAD:hussain
                        bat "kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Reaching at the end of pipeline...'
            // Add any post-build clean-up steps here
        }
        success {
            echo 'Build, test, and deployment completed successfully.'
        }
        failure {
            echo 'The build, test, or deployment failed.'
        }
    }
}