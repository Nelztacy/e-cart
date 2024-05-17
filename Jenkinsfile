pipeline {
    agent any
    
    tools {
        maven 'localMaven'
        jdk 'jdk 17'
        // Remove sonar tool type and directly specify the SonarScanner tool
        sonarScanner 'sonar-scanner' 
    }

    environment {
        // Use 'tool' method to define the SonarScanner tool location
        SCANNER_HOME= tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Nelztacy/e-cart.git'
            }
        }
        
        stage('Mvn Compile') {
            steps {
                sh 'mvn compile'
            }
        }
        
        stage('Mvn Unit Test') {
            steps {
                sh 'mvn test -DskipTests=true'
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv(credentialsId: 'SonarQube-Token', installationName: 'SonarQube') {
                sh "$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=ECART -Dsonar.projectName=ECART -Dsonar.java.binaries=."                }
            }
        }
    }
}