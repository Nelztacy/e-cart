pipeline {
    agent any
    
    tools {
        maven 'localMaven'
        jdk 'jdk 17'
        // No need to declare SonarQubeScanner here, we'll handle it in the environment block
    }

    environment {
        SCANNER_HOME = tool name: 'sonar-scanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
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
        
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sona-ecart') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.projectKey=ECART \
                          -Dsonar.projectName=ECART \
                          -Dsonar.java.binaries=. '''
                }
            }
        }
        
        stage('OWASP Dependencies Check') {
            steps {
        dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DP-Check'
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Artifact Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        
        stage('Deploy Artifacts to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'global-maven', jdk: 'jdk 17', maven: 'localMaven', mavenSettingsConfig: '', traceability: true) {
                sh "mvn deploy -DskipTests=true"
                       }
            }
        }
    }
}
