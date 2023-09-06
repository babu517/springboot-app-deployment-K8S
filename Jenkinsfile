pipeline {
    agent any
    environment {
        app = 'frontend'
        IMAGE_TAG = "frontend-${BUILD_NUMBER}"
        AWS_ACCESS_KEY_ID = credentials('AWS-CRED')
        AWS_SECRET_ACCESS_KEY = credentials('AWS-CRED')
        AWS_DEFAULT_REGION = 'ap-southeast-1'
        EKS_CLUSTER_NAME = 'sandboxeks1'
        SONAR_LOGIN = credentials('Sonar-Creds')
        DOCKERHUB_CREDENTIALS = credentials('docker-rama')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "origin/${env.BRANCH_NAME}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/babu517/springboot-app-deployment-K8S']]])
            }
        }
        stage('Code Build') {
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Compile and Run Sonar Analysis') {
            steps {
                sh "mvn clean verify sonar:sonar  \
            -Dsonar.projectKey=frontend \
            -Dsonar.host.url=http:182.18.184.80:9000 \
            -Dsonar.login=sqa_50885d4939be4dfc296b4d5af73a7c307287fec8"
            }
        }
        stage('Push to S3') {
            steps {
                sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID'
                sh 'aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY'
                sh 'aws configure set default.region $AWS_DEFAULT_REGION'
                sh 'aws s3 ls'
                sh 'aws s3 cp target/*.jar s3://eksfrontendapp/'
                }
            }
               
        stage('Build Image') {
            steps {
                sh 'docker build -t frontendapp-${app}:latest .'
            }
        }
        stage('Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
           }
        }
        stage('Push to DOCKER') {
            steps {
                sh 'docker tag frontendapp-${app} ramakrishna8254/week:v5'
                sh 'docker push ramakrishna8254/week:v5'
            }
        }
        stage('Deploying Docker Image to EKS') {
            steps {
                script {
                    sh '''
                        aws eks update-kubeconfig --name $EKS_CLUSTER_NAME
                        kubectl apply -f eks-deploy-k8s.yaml 
                    '''
                }
            }
        }
        
    }
}
