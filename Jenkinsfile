pipeline {
    agent any

    tools {
        nodejs 'Nodejs'
    }

    environment {
        DOCKER_IMAGE = "nitinkdocker18/react-nodejs-app"
        DOCKER_TAG = "${BUILD_NUMBER}"
        EC2_HOST = "ec2-user@43.205.253.25"
    }

    stages {

        stage('Increment Version') {
            steps {
                echo 'Incrementing version...'
                sh '''
                  cd my-app
                  npm version patch --no-git-tag-version
                  cd ../api
                  npm version patch --no-git-tag-version
                '''
            }
        }

        stage('Build') {
            steps {
                echo 'Building React app...'
                dir('my-app') {
                    sh 'npm install'
                    sh 'NODE_OPTIONS=--openssl-legacy-provider npm run build'
                }

                echo 'Building backend...'
                dir('api') {
                    sh 'npm install'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    sh '''
                        docker build -t $DOCKER_IMAGE:$DOCKER_TAG .
                        docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_IMAGE:latest
                    '''

                    withCredentials([usernamePassword(
                        credentialsId: 'nitinkdocker18',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh '''
                            echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                            
                            docker push $DOCKER_IMAGE:$DOCKER_TAG
                            docker push $DOCKER_IMAGE:latest
                            
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying...'
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ec2-ssh-key',
                    keyFileVariable: 'EC2_KEY'
                )]) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no -i $EC2_KEY $EC2_HOST "

                        docker pull $DOCKER_IMAGE:$DOCKER_TAG &&

                        docker stop app || true &&
                        docker rm app || true &&

                        # kill anything using port 3000
                        docker ps -q --filter publish=3000 | xargs -r docker stop &&
                        docker ps -aq --filter publish=3000 | xargs -r docker rm &&

                        docker run -d \
                          --name app \
                          --restart always \
                          -p 3000:3080 \
                          $DOCKER_IMAGE:$DOCKER_TAG
                        "
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up Docker...'
            sh 'docker image prune -f'
        }
    }
}