pipeline {

    agent any


    tools {
        maven 'maven3'
        jdk 'java17'
    }


    environment {

        DOCKER_IMAGE = "daneshkabade45/springboot-enterprise-app"

        NAMESPACE = "application"

        CONTAINER_NAME = "springboot-container"

        SERVICE_NAME = "springboot-service"

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

                sh '''

                echo "Building Spring Boot Application"

                mvn clean package -DskipTests

                '''

            }

        }



        stage('SonarQube Analysis') {

            steps {

                withSonarQubeEnv('sonarqube') {

                    sh '''

                    echo "Running SonarQube Scan"

                    mvn sonar:sonar \
                    -Dsonar.projectKey=springboot-enterprise-app \
                    -Dsonar.projectName=springboot-enterprise-app

                    '''

                }

            }

        }



        stage('SonarQube Quality Gate') {

            steps {

                timeout(time: 5, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true

                }

            }

        }





        stage('Docker Build') {

            steps {

                sh '''

                echo "Building Docker Image"


                docker build \
                -t $DOCKER_IMAGE:$BUILD_NUMBER .


                docker tag \
                $DOCKER_IMAGE:$BUILD_NUMBER \
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

                    echo "Docker Login"


                    echo $DOCKER_PASS | docker login \
                    -u $DOCKER_USER \
                    --password-stdin



                    docker push $DOCKER_IMAGE:$BUILD_NUMBER


                    docker push $DOCKER_IMAGE:latest


                    '''

                }

            }

        }






        stage('Blue Green Deployment') {


            steps {


                sh '''

                echo "Finding current production environment"


                CURRENT=$(kubectl get service $SERVICE_NAME \
                -n $NAMESPACE \
                -o jsonpath='{.spec.selector.version}')


                echo "Current Environment: $CURRENT"



                if [ "$CURRENT" = "blue" ]

                then

                    NEW_ENV="green"

                else

                    NEW_ENV="blue"

                fi



                echo "Deploying $NEW_ENV environment"



                kubectl set image \
                deployment/springboot-$NEW_ENV \
                $CONTAINER_NAME=$DOCKER_IMAGE:$BUILD_NUMBER \
                -n $NAMESPACE




                kubectl rollout status \
                deployment/springboot-$NEW_ENV \
                -n $NAMESPACE \
                --timeout=120s





                echo "Switching Traffic"



                kubectl patch service $SERVICE_NAME \
                -n $NAMESPACE \
                -p "{\\"spec\\":{\\"selector\\":{\\"app\\":\\"springboot\\",\\"version\\":\\"$NEW_ENV\\"}}}"




                echo "Traffic switched successfully"


                '''

            }

        }






        stage('Verify Deployment') {


            steps {


                sh '''

                echo "Checking Service"


                kubectl describe service \
                $SERVICE_NAME \
                -n $NAMESPACE



                echo "Checking Pods"


                kubectl get pods \
                -n $NAMESPACE \
                --show-labels



                echo "Testing Application"


                curl -I \
                http://k8s-applicat-springbo-ccdab34c95-1432777456.ap-south-1.elb.amazonaws.com \
                || true


                '''

            }

        }


    }





    post {


        success {

            echo "BLUE GREEN DEPLOYMENT SUCCESSFUL 🚀"

        }



        failure {

            echo "Pipeline failed"

            echo "Existing production environment remains active"

        }

    }

}

