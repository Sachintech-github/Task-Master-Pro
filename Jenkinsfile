pipeline {
    agent any
    
    tools {
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git-checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Sachintech-github/Task-Master-Pro.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs --format table -o fs-report.html .'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=TaskMaster \
                    -Dsonar.projectName=TaskMaster -Dsonar.java.binaries=target '''
                }
            }
        }
        
        stage('Build Application') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Publish To Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings', jdk: '', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
                    sh 'mvn deploy'
                }
            }
        }
        
        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker build -t sachintech/taskmaster:latest .'
                    }
                }
            }
        }
        
        stage('Scan Docker Images by Trivy') {
            steps {
                sh 'trivy image --format table -o image-report.html sachintech/taskmaster:latest'
            }
        }
        
        stage('Docker Image Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh 'docker push sachintech/taskmaster:latest'
                    }
                }
            }
        }
        
        stage('Deploy To K8s') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'sachin-eks13', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://FE3FF0B87CA23E5841EDC2E30174AF38.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl apply -f deployment-service.yml -n webapps'
                    sleep 30
                }
            }
        }
        
        stage('verify the deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'sachin-eks13', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://FE3FF0B87CA23E5841EDC2E30174AF38.gr7.ap-south-1.eks.amazonaws.com') {
                    sh 'kubectl get pods -n webapps'
                    sh 'kubectl get svc -n webapps'
                }
            }
        }
        
    }
}
