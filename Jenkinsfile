
pipeline {
    agent any

    tools {
        maven 'maven3'
        
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git Checkout') {
            steps {
                echo 'Cloning Git repository...'
                git branch: 'main', url: 'YOUR_GITHUB_REPO_URL' // <<< REPLACE THIS with your forked GitHub repository URL
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format HTML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Trivy FS Scan"){
            steps{
                sh "trivy fs --formattable -o fs-report.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSOnarQubeEnv('sonar'){
                    sh " $SCANNER_HOME/bin/sonarscanner -Dsonar.projectName=Multitier -Dsonar.projectKey=Multitier -Dsonar.java.binaries=target"
                }
            }
        }

        stage("Build"){
            steps{
                sh "mvn package -DskipTests=true"
            }
        }

        stage("Publish To Nexus"){
            steps{
                withMaven(globalMavenSettingsConfig:'settings-maven',jdk:''. maven:'maven3' , mavenSettingsConfig:'traceability:true){
                          sh "mvn deploy -DskipTests=true"
            }
        }

        stage('Docker Build Image') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'b289dc43-2ede-4bd0-95e8-75ca26100d8d', toolName: 'docker') { // Generated using syntax pipeline generator tool
                            sh "docker build -t username/bankapp:latest"
                       }
                   } 
            }
        }

        stage("Trivy Image Scan"){
            steps{
                sh "trivy image --formattable -o fs-report.html username/bankapp:latest"
            }
        }

        stage('Docker Push Image') {
            steps {
                   script {
                       withDockerRegistry(credentialsId: 'b289dc43-2ede-4bd0-95e8-75ca26100d8d', toolName: 'docker') { // Generated using syntax pipeline generator tool
                            sh "docker push username/bankapp:latest"
                       }
                   } 
            }
        }





        stage('Deploy To Kubernetes') {
            steps {
                   script {
                       withKubeConfig(caCertificate: '',clusterName:'devopsshack-cluster',contextName:'',credentialsid:'k8-token',namespace:'webapps', restrictKubeConfigAccess:flase, serverUrl:"https://*************************"){
                        sh "kubectl apply -f ds.yml -n webapps"
                        sleep 30
                       }
                       
                   } 
            }
        }


        stage('Verify Deployment') {
            steps {
                   script {
                       withKubeConfig(caCertificate: '',clusterName:'devopsshack-cluster',contextName:'',credentialsid:'k8-token',namespace:'webapps', restrictKubeConfigAccess:flase, serverUrl:"https://*************************"){
                        sh "kubectl get pods -n webapps"
                        sh "kubectl get svc -n webapps"
                        sleep 30
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
