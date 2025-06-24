
pipeline {
    agent any

    tools {
        nodejs 'nodejs-10'
        
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'Cloning Git repository...'
                git branch: 'main', url: 'YOUR_GITHUB_REPO_URL' // <<< REPLACE THIS with your forked GitHub repository URL
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies...'
                sh 'npm install'
            }
        }

        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
      

        stage('Docker Build and Push Image') {
            steps {
                echo 'Building and pushing Docker image...'
                script {
                    withDockerRegistry(toolName: 'docker') {
                        sh "docker build -t demonodejs ."
                        sh "docker tag demonodejs username/nodejs:latest ."
                        sh "docker push username/nodejs:latest"
                    }
                }
            }
        }

        stage('Deployment') {
            steps {
                echo 'Deploying application in Docker container...'
                script {
                    script {
                    withDockerRegistry(toolName: 'docker') {
                        sh "docker run -d --name demonodejs -p 8081:8081 username/nodejs:latest"
                    }
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
