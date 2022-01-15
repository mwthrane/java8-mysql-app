#!/usr/bin/env groovy

pipeline {
    agent any
    environment {
        ECR_REPO_URL = '664574038682.dkr.ecr.eu-west-3.amazonaws.com'
        IMAGE_REPO = "${ECR_REPO_URL}/java-app"
        IMAGE_NAME = "1.0-${BUILD_NUMBER}"
        CLUSTER_NAME = "my-cluster"
        CLUSTER_REGION = "eu-west-3"
        AWS_ACCESS_KEY_ID = credentials('jenkins_aws_access_key_id')
        AWS_SECRET_ACCESS_KEY = credentials('jenkins_aws_secret_access_key')
    }
    stages {
        stage('build app') {
            steps {
               script {
                   echo "building the application..."
                   sh './gradlew clean build'
               }
            }
        }
        stage('build image') {
            steps {
                script {
                    echo "building the docker image..."
                    sh "docker build -t ${IMAGE_REPO}:${IMAGE_NAME} ."
                    sh "aws ecr get-login-password --region ${CLUSTER_REGION} | docker login --username AWS --password-stdin ${ECR_REPO_URL}"
                    sh "docker push ${IMAGE_REPO}:${IMAGE_NAME}"
                }
            }
        }
        stage('deploy') {
            environment {
                APP_NAME = 'java-app'
                APP_NAMESPACE = 'my-app'
            }
            steps {
                script {
                    sh "aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${CLUSTER_REGION}"

                    // set variable values for db-secret data, by accessing the secret values, defined in Jenkins credentials as a "secret text" credentials type. We access them using the credentials id
                    db_user = credentials('db_user')
                    db_pass = credentials('db_pass')
                    db_name = credentials('db_name')
                    db_root_pass = credentials('db_root_pass')

                    DB_USER = sh(script: "echo -n ${db_user} | base64", returnStdout: true).trim()
                    DB_PASS = sh(script: "echo -n ${db_pass} | base64", returnStdout: true).trim()
                    DB_NAME = sh(script: "echo -n ${db_name} | base64", returnStdout: true).trim()
                    DB_ROOT_PASS = sh(script: "echo -n ${db_root_pass} | base64", returnStdout: true).trim()
                    
                    echo "${DB_USER}"
                    
                    echo 'deploying new release to EKS...'
                    sh 'envsubst < k8s-deployment/java-app-cicd.yaml | kubectl apply -f -'
                    sh 'envsubst < k8s-deployment/db-config-cicd.yaml | kubectl apply -f -'
                    sh 'envsubst < k8s-deployment/db-secret-cicd.yaml | kubectl apply -f -'

                    
                }
            }
        }
    }
}
