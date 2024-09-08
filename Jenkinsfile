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

        registry = 'shalnark/hello-spring'
        registryCredential = credentials('dockerhub-credentials') // Stored in Jenkins Credentials Manager
        dockerImage = ''
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
                    sh """
                        cd ${env.BASE_PATH}
                        dockerImage = docker.build registry + ":noVersion"
                    """.stripIndent().trim()
                    echo "Docker image hello-spring:NoVersion has been built."
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
                    docker.withRegistry( '', registryCredential ) {
                        dockerImage.push()
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
