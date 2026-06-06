pipeline {

    agent any

    tools {
        maven 'maven3'
    }

    environment {
        DOCKER_IMAGE = "daneshkabade45/springboot-enterprise-app"
        NAMESPACE = "application"
        DEPLOYMENT = "springboot-app"
        CONTAINER = "springboot-container"
    }

    stages {


        stage('Checkout Code') {
            steps {
                git branch: 'main',
                url: 'https://github.com/danesh854/springboot-enterprise-app.git'
            }
        }


        stage('Maven Build') {
            steps {
                sh '''
                echo "Building Spring Boot Application"
                mvn clean package
                '''
            }
        }


        stage('Docker Build') {
            steps {
                sh '''
                echo "Building Docker Image"

                docker build -t $DOCKER_IMAGE:${BUILD_NUMBER} .

                docker tag \
                $DOCKER_IMAGE:${BUILD_NUMBER} \
                $DOCKER_IMAGE:latest
                '''
            }
        }


        stage('Docker Push') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {

                    sh '''
                    echo "Login DockerHub"

                    echo $DOCKER_PASS | \
                    docker login \
                    -u $DOCKER_USER \
                    --password-stdin


                    echo "Pushing Version Image"

                    docker push \
                    $DOCKER_IMAGE:${BUILD_NUMBER}


                    echo "Pushing Latest Image"

                    docker push \
                    $DOCKER_IMAGE:latest
                    '''
                }
            }
        }



        stage('Deploy To EKS') {

            steps {

                sh '''

                echo "Deploying Application To EKS"


                kubectl set image \
                deployment/$DEPLOYMENT \
                $CONTAINER=$DOCKER_IMAGE:${BUILD_NUMBER} \
                -n $NAMESPACE



                echo "Saving deployment history"


                kubectl annotate deployment \
                $DEPLOYMENT \
                kubernetes.io/change-cause="Jenkins deployed image $DOCKER_IMAGE:${BUILD_NUMBER}" \
                -n $NAMESPACE \
                --overwrite



                echo "Checking rollout"


                kubectl rollout status \
                deployment/$DEPLOYMENT \
                -n $NAMESPACE

                '''

            }

        }



        stage('Verify Deployment') {

            steps {

                sh '''

                echo "Pods Status"

                kubectl get pods \
                -n $NAMESPACE


                echo "Current Running Image"

                kubectl describe deployment \
                $DEPLOYMENT \
                -n $NAMESPACE | grep Image

                '''

            }
        }

    }


    post {


        success {

            echo "Pipeline completed successfully 🚀"

        }



        failure {

            echo "Deployment failed. Rolling back..."

            sh '''

            kubectl rollout undo \
            deployment/$DEPLOYMENT \
            -n $NAMESPACE || true


            kubectl rollout status \
            deployment/$DEPLOYMENT \
            -n $NAMESPACE || true

            '''

        }

    }

}
