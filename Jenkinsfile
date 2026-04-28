pipeline {
    agent any

    environment {
        DOCKER_USER     = "srinivasu56"
        IMAGE_NAME      = "js-project-zomato"
        IMAGE_TAG       = "latest"
        CONTAINER_NAME  = "zomato-container"
        DOCKER_CREDS    = "docker-cred"
        NEXUS_CRED_ID  = "nexus-cred"
        SONAR_SCANNER = "sonar-server"
    }

    stages {

        stage('Checkout Code') {
            steps {
                checkout scmGit(
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[
                        credentialsId: 'git-cred',
                        url: 'https://github.com/Srinivasu2000/DevOps-Project-Zomato-Kastro.git'
                    ]]
                )
            }
        }

        // Build Node.js project
        stage('Build NPM Project') {
            steps {
                sh '''
                    echo "Building Node.js application..."
                    npm install
                    npm run build
                '''
            }
        }

stage('Upload NodeJS Artifacts to Nexus') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'nexus-cred',
            usernameVariable: 'NEXUS_USER',
            passwordVariable: 'NEXUS_PASS'
        )]) {
            sh '''
                echo "Uploading build artifacts to Nexus..."

                NEXUS_URL="http://65.0.182.83:8081/repository/zomato"

                for file in $(find build -type f); do
                    curl -u $NEXUS_USER:$NEXUS_PASS --upload-file $file $NEXUS_URL/$(basename $file)
                done
            '''
        }
    }
}       

        // Build Docker image
        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Building Docker image..."
                    docker build -t ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG} .
                '''
            }
        }

        // Run container locally (for testing)
        stage('Run Container (Local Test)') {
            steps {
                sh '''
                    echo "Running container locally..."
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d --name ${CONTAINER_NAME} -p 8082:8080 ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                '''
            }
        }

        // Push image to DockerHub
        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: "${DOCKER_CREDS}",
                        usernameVariable: 'DOCKER_HUB_USER',
                        passwordVariable: 'DOCKER_HUB_PASS'
                    )
                ]) {
                    sh '''
                        echo "Logging in to DockerHub..."
                        echo "$DOCKER_HUB_PASS" | docker login -u "$DOCKER_HUB_USER" --password-stdin
                        docker push ${DOCKER_USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    '''
                }
            }
        }



stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('sonar-server') {
            sh '''
                sonar-scanner \
                -Dsonar.projectKey=zomato \
                -Dsonar.sources=. \
                -Dsonar.host.url=http://65.0.182.83:9000 \
                -Dsonar.login=$SONAR_AUTH_TOKEN
            '''
        }
    }
}


        

        // Deploy to Kubernetes (EKS)
        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    echo "Deploying to Kubernetes..."
                    rm -rf ~/.kube/config
                    aws eks --region ap-south-1 update-kubeconfig --name mycluster
                    kubectl get nodes
                    kubectl apply -f Kubernetes/deploymentfile.yml
                    kubectl apply -f Kubernetes/service.yml
                '''
            }
        }
    }
}
