pipeline {
    agent {
        label 'docker_slave'
    }
    
    options {
        timestamps()
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    
    environment {
        DEBUG = 'true'
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '555108715761.dkr.ecr.us-east-1.amazonaws.com'
    }
    
    stages {
        stage('Debug Info') {
            steps {
                script {
                    echo "=== Debug Information ==="
                    echo "Job Name: ${env.JOB_NAME}"
                    echo "Build Number: ${env.BUILD_NUMBER}"
                    echo "Build ID: ${env.BUILD_ID}"
                    echo "Workspace: ${env.WORKSPACE}"
                    echo "Node Name: ${env.NODE_NAME}"
                    echo "========================"
                    sh 'echo "Current User: $(whoami)"'
                    sh 'echo "Current Directory: $(pwd)"'
                    sh 'echo "Docker Version:"; docker --version'
                    sh 'echo "AWS CLI Version:"; aws --version'
                }
            }
        }
        
        stage('Git Login') {
            steps {
                echo 'Configuring Git credentials...'
                script {
                    try {
                        withCredentials([usernamePassword(credentialsId: 'github_pat', usernameVariable: 'GIT_USERNAME', passwordVariable: 'GIT_PASSWORD')]) {
                            sh """
                                git config --global credential.helper store
                                git config --global user.email "jenkins@example.com"
                                git config --global user.name "Jenkins CI"
                            """
                            echo 'Git configured successfully!'
                        }
                    } catch (Exception e) {
                        echo "Git configuration failed with error: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
        
        stage('Docker Login') {
            steps {
                echo 'Logging into AWS ECR...'
                script {
                    try {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        """
                        echo 'Docker login successful!'
                    } catch (Exception e) {
                        echo "Docker login failed with error: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
        
        stage('Parallel Build') {
            parallel {
                stage('Build UI') {
                    steps {
                        echo 'Building UI Docker image...'
                        script {
                            try {
                                dir('ui') {
                                    echo 'Cloning UI repository...'
                                    git branch: 'master',
                                        credentialsId: 'github_pat',
                                        url: 'https://github.com/kalpeshkolap/usermgmtui.git'
                                    
                                    sh 'echo "Files in UI directory:"; ls -la'
                                    
                                    echo "Building Docker image with tag: ${ECR_REGISTRY}/ui:${BUILD_ID}"
                                    sh """
                                        docker build -t ${ECR_REGISTRY}/ui:${BUILD_ID} .
                                    """
                                    echo 'UI Docker build successful!'
                                    
                                    sh 'echo "Docker images:"; docker images | grep ui'
                                }
                            } catch (Exception e) {
                                echo "UI build failed with error: ${e.getMessage()}"
                                throw e
                            }
                        }
                    }
                }
                
                stage('Build API') {
                    steps {
                        echo 'Building API Docker image...'
                        script {
                            try {
                                dir('api') {
                                    echo 'Cloning API repository...'
                                    // Add your API repository details here
                                    git branch: 'master',
                                    credentialsId: 'github_pat',
                                    url: 'https://github.com/kalpeshkolap/usermgmt.git'
                                    
                                    echo 'API build placeholder - add your API repository'
                                    
                                
                                    sh 'echo "Files in API directory:"; ls -la'
                                    echo "Building Docker image with tag: ${ECR_REGISTRY}/api:${BUILD_ID}"
                                    sh """
                                        docker build -t ${ECR_REGISTRY}/api:${BUILD_ID} .
                                    """
                                    echo 'API Docker build successful!'
                                }
                            } catch (Exception e) {
                                echo "API build failed with error: ${e.getMessage()}"
                                throw e
                            }
                        }
                    }
                }
                
                stage('Build Auth') {
                    steps {
                        echo 'Building Auth Docker image...'
                        script {
                            try {
                                dir('auth') {
                                    echo 'Cloning Auth repository...'
                                    // Add your Auth repository details here
                                    // git branch: 'master',
                                    //     credentialsId: 'github_pat',
                                    //     url: 'https://github.com/your-repo/auth.git'
                                    
                                    echo 'Auth build placeholder - add your Auth repository'
                                    
                                    // Uncomment when ready:
                                    // sh 'echo "Files in Auth directory:"; ls -la'
                                    // echo "Building Docker image with tag: ${ECR_REGISTRY}/auth:${BUILD_ID}"
                                    // sh """
                                    //     docker build -t ${ECR_REGISTRY}/auth:${BUILD_ID} .
                                    // """
                                    // echo 'Auth Docker build successful!'
                                }
                            } catch (Exception e) {
                                echo "Auth build failed with error: ${e.getMessage()}"
                                throw e
                            }
                        }
                    }
                }
            }
        }
        
        stage('Push Images') {
            steps {
                echo 'Pushing Docker images to ECR...'
                script {
                    try {
                        sh """
                            docker push ${ECR_REGISTRY}/ui:${BUILD_ID}
                        """
                        echo 'UI image pushed successfully!'
                        
                        // Uncomment when other services are ready:
                        // sh """
                        //     docker push ${ECR_REGISTRY}/api:${BUILD_ID}
                        //     docker push ${ECR_REGISTRY}/auth:${BUILD_ID}
                        // """
                        // echo 'All images pushed successfully!'
                    } catch (Exception e) {
                        echo "Docker push failed with error: ${e.getMessage()}"
                        throw e
                    }
                }
            }
        }
    }
    
    post {
        success {
            echo '=== Pipeline completed successfully! ==='
            echo "Build Duration: ${currentBuild.durationString}"
            echo "Image: ${ECR_REGISTRY}/ui:${BUILD_ID}"
        }
        failure {
            echo '=== Pipeline failed! ==='
            echo "Failed Stage: ${env.STAGE_NAME}"
            echo "Build Duration: ${currentBuild.durationString}"
            sh 'echo "Docker processes:"; docker ps -a || true'
        }
        always {
            echo 'Cleaning up workspace...'
            cleanWs()
        }
    }
}
