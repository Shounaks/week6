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
                echo 'BUILD_STEP: TOOLS_USED: MAVEN, GRADLE'
            }
        }

        stage('Unit & Integration Test') {
            steps {
                echo 'TEST_STEP: Unit Tests'
                echo 'TEST_STEP: TOOLS_USED: MOCKITO'
                echo 'TEST_STEP: Integration Tests'
                echo 'TEST_STEP: TOOLS_USED: WIREMOCK'
            }
        }

        stage('Code Analysis Check') {
            steps {
                echo 'CODE_ANALYSIS_STEP: Check the quality of the code'
                echo 'CODE_ANALYSIS_STEP: TOOLS_USED: SONARQUBE, SNYK APP SCAN'
            }
        }

        stage('Security Scan') {
            steps {
                echo 'SECURITY_SCAN: Check the code for any SAST Vulnerability'
                echo 'SECURITY_SCAN: TOOLS_USED: SONARQUBE, SNYK CONTAINER SCAN, ZAP'
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo 'DEPLOY_TO_STAGING: Check the code for any SAST Vulnerability'
                echo 'DEPLOY_TO_STAGING: TOOLS_USED: Azure Kubernetes Server'
            }
        }

        stage('Integration Test on Staging') {
            steps {
                echo 'SECURITY_SCAN: Check the code for any SAST Vulnerability'
                echo 'SECURITY_SCAN: TOOLS_USED: FLYWAY, ZEPHYR, TEST_NG'
            }
        }

//         stage('Approval') {
//             steps {
//                 echo 'Hello World'
//                 sleep 10
//             }
//         }

        stage('Deploy to Production') {
            steps {
                echo "DEPLOY_STEP: Deploy the application to a testing environment specified by the environment variable ${env.TESTING_ENVIRONMENT}"
                echo "DEPLOY_STEP: TOOLS_USED: AKS, Amazon Elastic Container Service"
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

