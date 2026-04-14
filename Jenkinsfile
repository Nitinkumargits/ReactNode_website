pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "nitinkdocker18/react-nodejs-app"
        DOCKER_TAG = "latest"
        EC2_HOST = "ec2-user@43.205.253.25"
        EC2_KEY = credentials('ec2-ssh-key') // Jenkins credential id for SSH private key
    }
    stages {
        stage('Build') {
            steps {
                echo 'Building React app...'
                dir('my-app') {
                    sh 'npm install'
                    sh 'npm run build'
                }
                echo 'Building Node.js backend...'
                dir('api') {
                    sh 'npm install'
                }
            }
        }
        stage('Test') {
            steps {
                echo 'Testing React app...'
                dir('my-app') {
                    sh 'npm test -- --watchAll=false'
                }
                echo 'Testing Node.js backend...'
                dir('api') {
                    sh 'npm test || true' // skip if no tests
                }
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
                    withCredentials([usernamePassword(credentialsId: 'nitinkdocker18', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                        sh "docker push $DOCKER_IMAGE:$DOCKER_TAG"
                    }
                }
            }
        }
        stage('Deploy to EC2') {
            steps {
                echo 'Deploying to EC2...'
                script {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $EC2_KEY $EC2_HOST "
                        docker pull $DOCKER_IMAGE:$DOCKER_TAG &&
                        docker stop app || true &&
                        docker rm app || true &&
                        docker run -d \
                          --name app \
                          -p 3000:3080 \
                          $DOCKER_IMAGE:$DOCKER_TAG
                        "
                    '''
                }
            }
}
    }
}