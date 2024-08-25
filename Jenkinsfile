pipeline {
    agent any

    environment {
        DIRECTORY_PATH = 'source/'
        TESTING_ENVIRONMENT = 'dev'
        PRODUCTION_ENVIRONMENT = 'shounak bhalerao'
    }

    stages {
        stage('Build') {
            steps {
                echo 'BUILD_STEP: Fetch the source code from the directory path specified by the environment variable'
                echo 'BUILD_STEP: Compile the code and generate any necessary artifacts'
            }
        }

        stage('Test') {
            steps {
                echo 'TEST_STEP: Unit Tests'
                echo 'TEST_STEP: Integration Tests'
            }
        }

        stage('Code Quality Check') {
            steps {
                echo 'CODE_QUALITY_STEP: Check the quality of the code'
            }
        }


        stage('Deploy') {
            steps {
                echo "DEPLOY_STEP: Deploy the application to a testing environment specified by the environment variable ${env.TESTING_ENVIRONMENT}"
            }
        }

        stage('Approval') {
            steps {
//                 echo 'Hello World'
                sleep 10
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "PRODUCTION_STEP: Deployed code situated at ${env.DIRECTORY_PATH}, to the following environment: ${env.PRODUCTION_ENVIRONMENT}"
            }
        }
    }
    post{
        success {
            emailext body: '[SUCCESS] Jenkins Week 6 Activity : Build Success',
                subject: '🟢 Jenkins Build | Test Subject',
                attachLog: true,
                to: 'shounakbhalerao777@gmail.com'
        }
        failure {
            emailext body: '[FAILURE] Jenkins Week 6 Activity : Build Success',
                subject: '🔴 Jenkins Build | Test Subject',
                attachLog: true,
                to: 'shounakbhalerao777@gmail.com'
        }
    }
}

