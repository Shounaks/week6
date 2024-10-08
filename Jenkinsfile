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

        DOCKER_CREDENTIALS_ID = 'dockerhub-credentials' // Stored in Jenkins Credentials Manager

        IMAGE_NAME = 'shalnark/myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        DEPLOY_NAME = 'deploy-hello-world'
        SSH_CREDENTIALS_ID = 'azure-vm'
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
                script {
                    def AZURE_VM_IP = input message: 'Enter the IP address of the Azure VM:',
                        parameters: [string(defaultValue: '0.0.0.0', description: 'Azure VM IP', name: 'IP_ADDRESS')]
                    sshagent (credentials: [SSH_CREDENTIALS_ID]) {
                        sh """
                        ssh -o StrictHostKeyChecking=no azureuser@${AZURE_VM_IP} 'sudo chmod 666 /var/run/docker.sock && docker pull ${IMAGE_NAME}:latest && docker stop ${DEPLOY_NAME} || true && docker rm ${DEPLOY_NAME} || true && docker run -d --name ${DEPLOY_NAME} -p 8080:8080 ${IMAGE_NAME}:latest'
                        """
                    }
                }
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    sh "docker stop ${DEPLOY_NAME}"
                    sh "docker system prune -f"
                    sh "docker rmi ${IMAGE_NAME}:${DOCKER_TAG}"
                }
            }
        }
    }
}