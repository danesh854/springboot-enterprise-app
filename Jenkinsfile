pipeline {

    agent any

    tools {
        maven 'maven3'
        jdk 'java17'
    }


    environment {
        DOCKER_IMAGE = "daneshkabade45/springboot-enterprise-app"
    }


    stages {


        stage('Checkout Code') {

            steps {

                git branch: 'main',
                credentialsId: 'github-creds',
                url: 'https://github.com/danesh854/springboot-enterprise-app.git'

            }
        }



        stage('Maven Build') {

            steps {

                sh 'mvn clean package -DskipTests'

            }
        }



        stage('Docker Build') {

            steps {

                sh '''

                docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .

                docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest

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

                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin


                    docker push $DOCKER_IMAGE:$BUILD_NUMBER

                    docker push $DOCKER_IMAGE:latest

                    '''

                }
            }
        }




        stage('Deploy To EKS') {

            steps {

                sh '''

                kubectl set image deployment/springboot-app \
                springboot-container=$DOCKER_IMAGE:$BUILD_NUMBER \
                -n application


                kubectl rollout status deployment/springboot-app \
                -n application

                '''

            }
        }


    }


    post {

        success {
            echo "Pipeline completed successfully"
        }


        failure {
            echo "Pipeline failed"
        }

    }

}
