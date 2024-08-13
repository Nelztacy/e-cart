pipeline {
    agent any

    tools {
        maven 'localMaven'
        jdk 'jdk17'
    }

    environment {
        SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
        // Remove Nexus-related variables
        ARTIFACT_VERSION = "${BUILD_NUMBER}"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Nelztacy/e-cart.git'
            }
        }

        stage('Mvn Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Mvn Unit Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }

        stage('OWASP Dependencies Check') {
            steps {
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Artifact Build') {
            steps {
                sh "mvn package -DskipTests=true"
                sh 'ls -l target'
            }
        }

        stage('Build & Tag Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Dockerhub_Cred', toolName: 'Docker') {
                        sh "docker build -t nelzone/ecart:latest -f docker/Dockerfile ."
                    }
                }
            }
        }

        stage('Trivy Scan') {
            steps {
                sh "trivy image nelzone/ecart:latest > trivy-report.txt "
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'Dockerhub_Cred', toolName: 'Docker') {
                        sh "docker push nelzone/ecart:latest"
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                withKubeConfig(caCertificate: '', clusterName: 'kubernetes', contextName: 'kubernetes', credentialsId: 'kubernetes-credentials', namespace: 'dev', restrictKubeConfigAccess: false, serverUrl: 'https://vm2:6443') {
                    sh "kubectl apply -f deploymentservice.yml"
                    sh "kubectl get svc -n jenkins"
                }
            }
        }
    }
}
