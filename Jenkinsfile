pipeline {
        options {
            skipDefaultCheckout()
        }
        stage('Check for [skip ci]') {
            steps {
                script {
                    def commitMsg = sh(script: 'git log -1 --pretty=%B', returnStdout: true).trim()
                    if (commitMsg.contains('[skip ci]')) {
                        echo 'Found [skip ci] in commit message. Exiting pipeline early to prevent loop.'
                        currentBuild.result = 'SUCCESS'
                        // Exit pipeline
                        return
                    }
                }
            }
        }
    agent any

    tools {
        nodejs 'Nodejs'
    }

    environment {
        DOCKER_IMAGE = "nitinkdocker18/react-nodejs-app"
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
                                    cd ..
                                    git config user.email "ci-bot@example.com"
                                    git config user.name "ci-bot"
                                    git add my-app/package.json api/package.json
                                    git commit -m "ci: increment version [skip ci]" || echo "No changes to commit"
                                    git push origin HEAD:master || echo "No changes to push"
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
                dir(env.WORKSPACE) {
                    script {
                        // Get versions from backend and frontend package.json
                        def backendVersion = sh(script: "node -p -e \"require('./api/package.json').version\"", returnStdout: true).trim()
                        def frontendVersion = sh(script: "node -p -e \"require('./my-app/package.json').version\"", returnStdout: true).trim()
                        def combinedTag = backendVersion + "-fe" + frontendVersion
                        env.DOCKER_TAG = combinedTag
                        sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
                        sh "docker tag $DOCKER_IMAGE:$DOCKER_TAG $DOCKER_IMAGE:latest"
                        withCredentials([usernamePassword(credentialsId: 'nitinkdocker18', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh '''
                                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                                docker push "$DOCKER_IMAGE:$DOCKER_TAG"
                                docker push "$DOCKER_IMAGE:latest"
                                docker logout
                            '''
                        }
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo 'Deploying...'
                withCredentials([sshUserPrivateKey(credentialsId: 'ec2-ssh-key', keyFileVariable: 'EC2_KEY')]) {
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