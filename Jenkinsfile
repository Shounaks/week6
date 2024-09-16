pipeline {
    agent any

    environment {
        REPO_URL = 'https://github.com/Shounaks/hello-spring.git'
        BASE_PATH = 'hello-spring/'
        TARGET_PATH = 'target/'
        DIRECTORY_PATH = 'source/'
        TESTING_ENVIRONMENT = 'dev'
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_LOGIN = credentials('sonarqube-token') // Stored in Jenkins Credentials Manager
        PRODUCTION_ENVIRONMENT = 'shounak bhalerao'
        OWNER = 'shounak bhalerao'

//         registry = 'shalnark/hello-spring'
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials' // Stored in Jenkins Credentials Manager
        IMAGE_NAME = 'shalnark/myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'

        DEPLOY_NAME = 'deploy-hello-world'

//         Release
        AZURE_SUBSCRIPTION_ID = '1d4c8195-4ad3-4a8b-b2c1-ec4325e334ea'
        AZURE_TENANT_ID = 'd02378ec-1688-46d5-8540-1c28b5f470f6'
        AZURE_CLIENT_ID = '3288c6ce-5f83-402a-9002-2ac9529c92a3'
        AZURE_CLIENT_SECRET = credentials('azure-client-secret')
        DOCKER_IMAGE = 'shalnark/myapp:latest'
        RESOURCE_GROUP = 'PIT-HD'
        CONTAINER_NAME = 'spring-hello-world'
        DNS_LABEL = 'myapp'
        LOCATION = 'southindia'
    }

    stages {
    // REFERENCES USED
    // Reference: https://www.jenkins.io/doc/tutorials/build-a-java-app-with-maven/
    // https://www.geeksforgeeks.org/how-to-integrate-sonarqube-with-jenkins/
    // https://hackernoon.com/how-to-make-docker-images-with-jenkins-pipelines-xsh3uza

        stage('Build') {
            steps {
                echo 'BUILD: Cleaning in progress'
                sh "rm -rf ${env.BASE_PATH}"

                echo 'BUILD: Verification'
                sh "git --version"
                sh "docker --version"
                sh "mvn --version"

                echo 'BUILD: Cloning Repository to local and trying to create an executable'
                sh "git clone ${env.REPO_URL}"
                sh """
                    cd ${env.BASE_PATH}
                    mvn -B -DskipTests clean package
                """.stripIndent().trim()
            }
        }

        stage('Test') {
            steps {
                echo 'TEST_STEP: Unit Tests'
                sh """
                    cd ${env.BASE_PATH}
                    mvn test
                """.stripIndent().trim()
            }
            post{
                always {
                    junit "${env.BASE_PATH}target/surefire-reports/*.xml"
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    script {
                        sh """
                            cd ${env.BASE_PATH}
                            mvn sonar:sonar -Dsonar.host.url=${env.SONAR_HOST_URL} -Dsonar.login=${env.SONAR_LOGIN}
                        """.stripIndent().trim()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'BUILD: DOCKER IMAGE'
                script{
                    dir('hello-spring') {
                        def appImage = docker.build("${IMAGE_NAME}:${DOCKER_TAG}")
                        echo "Docker image has been built!"
                        // Push the Docker image to DockerHub
                        docker.withRegistry(DOCKER_REGISTRY, DOCKER_CREDENTIALS_ID) {
                            appImage.push("${DOCKER_TAG}") // Push the image with build number tag
                            appImage.push("latest") // Optionally push the image with 'latest' tag
                        }
                    }
                }
                echo "DEPLOY_STEP: Pushed Image To DockerHub"
                script{
                    sh "docker run -d -p 9090:8080 --name ${DEPLOY_NAME} ${IMAGE_NAME}:${DOCKER_TAG}"
                    input message: 'Completed with Release?',
                        ok: 'Yes',
                        timeout: 300 //5 mins
                        timeoutMessage: 'Approval timed out. Proceeding to cleanup'
                }
            }
        }

        stage('Release') {
            steps {
                echo "RELEASE_STEP: Azure Login"
                sh """az login --service-principal \
                        --username ${AZURE_CLIENT_ID} \
                        --password ${AZURE_CLIENT_SECRET} \
                        --tenant ${AZURE_TENANT_ID}
                    az group create --name ${RESOURCE_GROUP} --location ${LOCATION}
                    az container create \
                        --resource-group ${RESOURCE_GROUP} \
                        --name ${CONTAINER_NAME} \
                        --image ${DOCKER_IMAGE} \
                        --cpu 1 \
                        --memory 1 \
                        --registry-login-server docker.io \
                        --dns-name-label ${DNS_LABEL} \
                        --ports 80
                    az container show --resource-group ${RESOURCE_GROUP} --name ${CONTAINER_NAME} --output table
                    echo "Deployment complete. Access your container at http://${DNS_LABEL}.${LOCATION}.azurecontainer.io"
                """
                script{
                    input message: 'Completed with Release?',
                        ok: 'Yes',
                        timeout: 300 //5 mins
                        timeoutMessage: 'Approval timed out. Proceeding to cleanup'
                }
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    sh "az group delete --name $RESOURCE_GROUP --yes --no-wait"
                    sh "docker stop ${DEPLOY_NAME}"
                    sh "docker system prune -f"
                    sh "docker rmi ${IMAGE_NAME}:${DOCKER_TAG}"
                }
            }
        }
    }
//     post{
//         success {
//             emailext body: '[SUCCESS] Jenkins Week 6 Activity : Build Success',
//                 subject: '🟢 Jenkins Build | Test Subject',
//                 attachLog: true,
//                 to: 'shounakbhalerao777@gmail.com'
//         }
//         failure {
//             emailext body: '[FAILURE] Jenkins Week 6 Activity : Build Success',
//                 subject: '🔴 Jenkins Build | Test Subject',
//                 attachLog: true,
//                 to: 'shounakbhalerao777@gmail.com'
//         }
//         always{
//             script {
//                 // This requires a SonarQube webhook setup which cant be done on
//                 // the same server as jenkins so skipping this part as for now.
//                 // It doesnt affect our functionality
//                 // Optionally, display SonarQube quality gate status after analysis
//                 // waitForQualityGate abortPipeline: true
//             }
//         }
//     }
}
