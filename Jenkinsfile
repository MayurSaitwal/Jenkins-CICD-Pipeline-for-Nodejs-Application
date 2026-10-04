pipeline {
    agent { label 'project' }

    stages {

        stage('Code') {
            steps {
                echo 'This is the cloning of the code'

                git branch: 'main',
                    url: 'https://github.com/MayurSaitwal/End-To-End-CICD-Pipeline-With-Kubernetes-Monitoring.git'

                echo 'Code cloned successfully'
            }
        }

        stage('Build') {
            steps {
                echo 'This is the building of code'

                sh '''
                    docker build -t nodes-js-app:latest .
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'This is the testing phase'
            }
        }

        stage('Deploy with Docker') {
            steps {
                echo 'Deploying application using Docker'

                sh '''
                    docker stop todo-app || true
                    docker rm todo-app || true

                    docker run -d \
                        --name todo-app \
                        -p 8000:8000 \
                        nodes-js-app:latest
                '''
            }
        }

        stage('Push Image to Docker Hub') {
            steps {
                echo 'Pushing image to Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin

                        docker tag nodes-js-app:latest $DOCKER_USERNAME/nodes-js-app:latest

                        docker push $DOCKER_USERNAME/nodes-js-app:latest

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy To Kubernetes') {
            steps {
                echo 'This is the deployment stage of Kubernetes'

                sh '''
                    kubectl apply -f k8s/namespace.yml

                    kubectl apply -f k8s/deployment.yml
                    kubectl apply -f k8s/service.yml

                    kubectl rollout restart deployment/node-app-deployment -n node-app

                    kubectl rollout status deployment/node-app-deployment -n node-app
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                echo 'Checking Kubernetes deployment'

                sh '''
                    kubectl get deployments -n node-app
                    kubectl get pods -n node-app
                    kubectl get services -n node-app
                '''
            }
        }
    }
}
