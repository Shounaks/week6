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
        IMAGE_NAME = 'myapp'
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        DOCKER_REGISTRY = 'https://index.docker.io/v1/shalnark/hello-spring'
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

                echo 'BUILD: DOCKER IMAGE'
                script{
                    dir('hello-spring') {
                        def appImage = docker.build("${IMAGE_NAME}:${DOCKER_TAG}")
                        echo "Docker image hello-spring:NoVersion has been built."
                    }
                }
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
                echo "DEPLOY_STEP: Push Image To DockerHub"
                script {
                    // Push the Docker image to DockerHub
                    docker.withRegistry(DOCKER_REGISTRY, DOCKER_CREDENTIALS_ID) {
                        appImage.push("${DOCKER_TAG}") // Push the image with build number tag
                        appImage.push("latest") // Optionally push the image with 'latest' tag
                    }
                }
            }
        }

        stage('Release') {
            steps {
                echo "DEPLOY_STEP: do nothing"
            }
        }
        stage('Cleanup') {
            steps {
                script {
                    sh "docker rmi ${registry} :noVersion"
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
