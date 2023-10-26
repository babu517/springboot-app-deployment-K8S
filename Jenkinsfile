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
    }

    
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: "origin/${env.BRANCH_NAME}"]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: 'github', url: 'https://github.com/babu517/springboot-app-deployment-K8S']]])
            }
        }
        stage('Build and Test') {
            steps {
                
                script {
                    // Get the current branch name
                    currentBranch = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true).trim()

                    // Get the current repository URL
                    currentRepoUrl = env.BUILD_URL // You can use a different environment variable for the repository URL

                    if (currentBranch == 'main' && currentRepoUrl == 'https://github.com/babu517/springboot-app-deployment-K8S.git') {
                        echo 'Building and testing for UAT branch in the specific repository'
                        // Add your build and test commands for UAT branch here
                    } else if (currentBranch == 'dev' && currentRepoUrl == 'https://github.com/babu517/springboot-app-deployment-K8S.git') {
                        echo 'Building and testing for Dev branch in the specific repository'
                        // Add your build and test commands for Dev branch here
                    } else {
                        echo 'Skipping build for other branches or repositories'
                    }
                }
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
               
        stage('Snyk Test') {
            steps {
                echo 'Snyk Testing...'
                snykSecurity (
                    projectName: 'babu517', 
                    snykInstallation: 'snyk@latest', 
                    snykTokenId: 'Snyk',
                    failOnIssues: false
                )
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t frontendapp-${app} .'
            }
        }
        stage('Push to ECR') {
            steps {
                sh 'aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 490167669940.dkr.ecr.ap-southeast-1.amazonaws.com'
                sh "docker tag frontendapp-${app}:latest  490167669940.dkr.ecr.ap-southeast-1.amazonaws.com/eks-frontend-app-deployment:${app}-${BUILD_NUMBER}"
                sh "docker push 490167669940.dkr.ecr.ap-southeast-1.amazonaws.com/eks-frontend-app-deployment:${app}-${BUILD_NUMBER}"
            }
        }
        stage('Deploying ECR Image to EKS') {
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
