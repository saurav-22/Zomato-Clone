properties([
    parameters([
        choice(
            choices: ['dev', 'stage', 'prod'],
            description: 'Select deployment environment',
            name: 'DEPLOYMENT_ENV'
        )
    ])
])
pipeline {
    agent any
    tools {
        jdk 'jdk24'
        nodejs 'NodeJs16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        AWS_REGION = 'ap-south-1' // Add your AWS region
        ECR_REGISTRY = 'XXXXX' // Add your ECR registry URI
        ECR_REPOSITORY = "project/zomato" // Add your ECR repository name
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            } 
        }
        stage('Checkout from Git') {
            steps {
                script {
                    try {
                        git branch: 'main', url: 'https://github.com/Aj7Ay/Zomato-Clone.git'
                    } catch (Exception e) {
                        echo "Git checkout failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Git checkout failed")
                    }
                }
            }
        }
        
        stage('Sonarqube Analysis') {
            steps {
                script {
                    try {
                        withSonarQubeEnv('sonar-server') {
                            sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=zomato -Dsonar.projectKey=zomato"
                        }
                    } catch (Exception e) {
                        echo "SonarQube analysis failed: ${e.message}"
                        // Continue pipeline but mark as unstable
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    try {
                        waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-Token'
                    } catch (Exception e) {
                        echo "Quality Gate failed: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
        
        stage('Build Image and Push to ECR') {
            steps {
                script {
                    try {
                        sh '''
                            aws ecr get-login-password --region ${AWS_REGION} | \
                            docker login --username AWS --password-stdin ${ECR_REGISTRY}

                            docker build -t ${ECR_REPOSITORY}:latest .
                            docker tag ${ECR_REPOSITORY}:latest ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                            docker push ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                        '''
                    } catch (Exception e) {
                        echo "Docker build/push failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Docker operations failed")
                    }
                }
            }
        }

        stage("TRIVY Security Scan") {
            steps {
                script {
                    try {
                        sh "trivy image ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest > trivy-scan-results.txt"
                        
                        // Check for critical vulnerabilities
                        def trivyResults = readFile('trivy-scan-results.txt')
                        if (trivyResults.contains('CRITICAL') || trivyResults.contains('HIGH')) {
                            echo "Critical/High vulnerabilities found!"
                            currentBuild.result = 'UNSTABLE'
                        }
                    } catch (Exception e) {
                        echo "Trivy scan failed: ${e.message}"
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                script {
                    try {
                        sh """
                            docker run -d \
                                --name zomato-${DEPLOYMENT_ENV} \
                                -p 3000:3000 \
                                -e NODE_ENV=${DEPLOYMENT_ENV} \
                                ${ECR_REGISTRY}/${ECR_REPOSITORY}:latest
                        """
                    } catch (Exception e) {
                        echo "Deployment failed: ${e.message}"
                        currentBuild.result = 'FAILURE'
                        error("Deployment failed")
                    }
                }
            }
        }
    }
    post {
        always {
            echo "Pipeline execution completed with status: ${currentBuild.result}"
            archiveArtifacts artifacts: 'trivy-scan-results.txt', allowEmptyArchive: true
        }
        success {
            echo "SUCCESS: ${JOB_NAME} #${BUILD_NUMBER} deployed to ${DEPLOYMENT_ENV}"
        }
        failure {
            echo "FAILED: ${JOB_NAME} #${BUILD_NUMBER} - ${currentBuild.currentResult}"
        }
        unstable {
            echo "UNSTABLE: ${JOB_NAME} #${BUILD_NUMBER} - Quality/Security issues detected"
        }
    }
}
