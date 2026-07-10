// Project 5 (combined): Automated, gated CI plus CD to Kubernetes (EKS).
//
// Every push triggers the pipeline. It must pass four gates (coverage,
// SonarQube Quality Gate, dependency scan, image scan) before the image
// is pushed to ECR and rolled out to an EKS cluster.
//
// EDIT the values marked below.

pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        // ----- EDIT THESE -----
        AWS_REGION  = 'us-east-1'
        ECR_ACCOUNT = '325879634650'
        ECR_REPO    = 'cicd-project-3'
        EKS_CLUSTER = 'cicd-cluster'
        // ----------------------

        IMAGE_TAG    = "${BUILD_NUMBER}"
        ECR_REGISTRY = "${ECR_ACCOUNT}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE        = "${ECR_REGISTRY}/${ECR_REPO}"

        SONAR_HOST   = 'http://sonarqube:9000'

        AWS_CREDS    = credentials('aws-ecr')
        SONAR_TOKEN  = credentials('sonar-token')
    }

    stages {
        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Build') {
            steps { dir('app') { sh 'mvn -B -ntp clean compile' } }
        }

        stage('Test + Coverage') {
            steps { dir('app') { sh 'mvn -B -ntp test' } }
            post { always { junit 'app/target/surefire-reports/*.xml' } }
        }

        stage('Code Analysis (SonarQube)') {
            steps {
                dir('app') {
                    sh '''
                        mvn -B -ntp sonar:sonar \
                          -Dsonar.host.url=${SONAR_HOST} \
                          -Dsonar.token=${SONAR_TOKEN} \
                          -Dsonar.projectKey=cicd-project-5 \
                          -Dsonar.projectName=cicd-project-5 \
                          -Dsonar.qualitygate.wait=true
                    '''
                }
            }
        }

        stage('Dependency Scan (Trivy)') {
            steps {
                sh 'trivy fs --scanners vuln --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 --no-progress app'
            }
        }

        stage('Package') {
            steps { dir('app') { sh 'mvn -B -ntp package -DskipTests' } }
        }

        stage('Docker Build') {
            steps {
                dir('app') {
                    sh 'docker build -t ${IMAGE}:${IMAGE_TAG} -t ${IMAGE}:latest .'
                }
            }
        }

        stage('Image Scan (Trivy)') {
            steps {
                sh 'trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 --no-progress ${IMAGE}:${IMAGE_TAG}'
            }
        }

        stage('Push to ECR') {
            steps {
                sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_CREDS_USR
                    export AWS_SECRET_ACCESS_KEY=$AWS_CREDS_PSW
                    aws ecr describe-repositories --region ${AWS_REGION} --repository-names ${ECR_REPO} \
                      || aws ecr create-repository --region ${AWS_REGION} --repository-name ${ECR_REPO}
                    aws ecr get-login-password --region ${AWS_REGION} \
                      | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                    docker push ${IMAGE}:${IMAGE_TAG}
                    docker push ${IMAGE}:latest
                '''
            }
        }

        stage('Deploy to EKS') {
            steps {
                sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_CREDS_USR
                    export AWS_SECRET_ACCESS_KEY=$AWS_CREDS_PSW
                    aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
                    sed "s|IMAGE_PLACEHOLDER|${IMAGE}:${IMAGE_TAG}|g" k8s/deployment.yaml | kubectl apply -f -
                    kubectl apply -f k8s/service.yaml
                    kubectl rollout status deployment/cicd-app --timeout=180s
                    kubectl get service cicd-app
                '''
            }
        }
    }

    post {
        success {
            echo "All gates passed. Deployed ${IMAGE}:${IMAGE_TAG} to ${EKS_CLUSTER}."
            echo "App URL: kubectl get service cicd-app (EXTERNAL-IP).  SonarQube: http://<your-host>:9000/"
        }
        failure {
            echo 'A stage or gate failed. Read the first red error above to see which one.'
        }
    }
}
