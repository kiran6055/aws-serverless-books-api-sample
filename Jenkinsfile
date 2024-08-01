pipeline {
    agent any 

    environment {
        AWS_ACCOUNT_ID = credentials('aws-account-id')
        REGION = "ap-south-1"
        AWS_Cred = "aws-credentials"
    }

    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod', 'uat'], description: 'Select the environment you want to apply')
        choice(name: 'BRANCH', choices: ['master', 'dev', 'uat'], description: 'Select the branch to apply')
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    cleanWs()
                    git credentialsId: 'kiranterraform', url: 'https://github.com/kiran6055/aws-serverless-books-api-sample.git', branch: "${params.BRANCH}"
                }
            }
        }

        stage('Validation Template') {
            steps {
                script {
                    sh 'curl -L -o aws-sam-cli-linux-x86_64.zip https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip'
                    sh 'unzip aws-sam-cli-linux-x86_64.zip -d sam-installation'
                    sh 'sudo ./sam-installation/install --update'
                    sh 'sam --version'
                    sh 'curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"'
                    sh 'unzip awscliv2.zip'
                    sh './aws/install'
                    sh 'cd SAMBackEnd && sam validate'
                    sh 'aws cloudformation validate-template --template-body file://template.yml'
                }
            }
        }

        stage('npm build') {
            steps {
                script {
                    sh 'npm install'
                    sh 'npm run setup-deps'
                    sh 'cd SAMBackEnd && npm install'
                    sh 'cd SAMBackEnd && NODE_OPTIONS=--max-old-space-size=8192 npm test'
                }
            }
        }

//        stage('Static Code Analysis') {
//            steps {
//                withCredentials([string(credentialsId: 'sonarqubetoken', variable: 'SONAR_AUTH_TOKEN')]) {
//                    sh "/opt/sonar-scanner/bin/sonar-scanner -Dsonar.login=$SONAR_AUTH_TOKEN -Dsonar.host.url=http://13.201.7.245:9000 -Dsonar.projectKey=${APP_NAME} -Dsonar.qualitygate.wait=true"
//                }
//            }
//        }

        stage('Deployment') {
            steps {
                script {
                    sh './deployment.sh'
                }
            }
        }
    }

}

